# Common FastAPI Error Prevention Patterns

Ten specific errors that cause production incidents in FastAPI applications. Each entry documents the symptom, root cause, prevention strategy with code, and detection method.

---

## 1. Form Data Metadata Loss

**Symptom:** OpenAPI documentation at `/docs` shows missing descriptions, examples, and titles for form fields. The Swagger UI renders form inputs without helpful context, and generated client libraries lack parameter documentation.

**Root Cause:** When `File()` and `Form()` parameters are combined in the same endpoint, FastAPI encodes the request body as `multipart/form-data`. The OpenAPI specification has limited support for per-field metadata in multipart encodings. FastAPI silently drops `description`, `example`, and `title` kwargs from `Form()` when a `File()` parameter is present in the same function signature.

**Prevention:** Separate file upload endpoints from form data endpoints when metadata matters. If they must coexist, document the limitation explicitly and provide metadata through the endpoint docstring or a dedicated schema.

```python
# Bad — metadata silently dropped
@app.post("/upload")
async def upload(
    file: UploadFile = File(...),
    name: str = Form(..., description="This description disappears"),
):
    ...

# Good — separate concerns
class DocumentMeta(BaseModel):
    name: str = Field(..., description="Document display name")
    category: str = Field(..., description="Filing category")

@app.post("/documents/metadata")
async def set_metadata(meta: DocumentMeta):
    ...

@app.post("/documents/{doc_id}/file")
async def upload_file(doc_id: int, file: UploadFile = File(...)):
    ...
```

**Detection:** Search for endpoints combining both parameter types.

```bash
grep -n "File(" app/ -r | xargs -I{} grep -l "Form(" {}
```

---

## 2. BackgroundTasks Overwrite

**Symptom:** Only the last dependency's background tasks execute. Earlier dependencies appear to add tasks successfully, but those tasks never run. No error is raised — the tasks silently vanish.

**Root Cause:** FastAPI creates a new `BackgroundTasks` instance for each dependency that declares it as a parameter. When multiple dependencies each receive their own `BackgroundTasks` object, only the instance from the final dependency in the resolution chain gets attached to the response. Tasks added to earlier instances are garbage collected.

**Prevention:** Declare `BackgroundTasks` only in the endpoint function itself, never in sub-dependencies. Pass the endpoint's `BackgroundTasks` instance to dependencies explicitly if they need to schedule work.

```python
# Bad — tasks from dep_a are lost
async def dep_a(bg: BackgroundTasks):
    bg.add_task(send_notification)  # never executes

async def dep_b(bg: BackgroundTasks):
    bg.add_task(update_cache)  # this one runs

@app.post("/action", dependencies=[Depends(dep_a), Depends(dep_b)])
async def action():
    ...

# Good — single BackgroundTasks owner
async def dep_a():
    return {"notify": True}

@app.post("/action")
async def action(bg: BackgroundTasks, info: dict = Depends(dep_a)):
    if info["notify"]:
        bg.add_task(send_notification)
    bg.add_task(update_cache)
```

**Detection:** Search for `BackgroundTasks` declared as a dependency parameter outside endpoint functions.

```bash
grep -rn "def .*BackgroundTasks" app/ | grep -v "@app\."
```

---

## 3. Optional Form Field Regression

**Symptom:** Form fields that accepted `None` in Pydantic v1 now raise 422 validation errors after upgrading to Pydantic v2. The error message says the field is required even though it was annotated as `Optional`.

**Root Cause:** Pydantic v2 changed how `Optional` interacts with FastAPI's `Form()`. In v1, `Optional[str] = Form(None)` correctly made the field optional. In v2, the `None` default inside `Form()` is not always propagated correctly, especially when `Optional` is used from `typing`. The v2 engine treats the type annotation and the default value independently.

**Prevention:** Use the modern union syntax with an explicit `default` keyword argument in `Form()`.

```python
# Fragile — breaks across Pydantic versions
from typing import Optional

@app.post("/search")
async def search(
    query: Optional[str] = Form(None),  # may raise 422 in v2
):
    ...

# Robust — works in both versions
@app.post("/search")
async def search(
    query: str | None = Form(default=None),
):
    ...
```

**Detection:** Find all Optional form fields that may be affected.

```bash
grep -rn "Optional.*Form" app/
```

---

## 4. Json Type with Form

**Symptom:** Sending a JSON string inside a form field annotated with `Json[dict]` produces a 422 validation error. The error message references an unexpected string where an object was expected, even though the raw value is valid JSON.

**Root Cause:** FastAPI's `Json` type annotation is designed for JSON request bodies (`application/json`), not for form data (`multipart/form-data` or `application/x-www-form-urlencoded`). When a form field arrives, FastAPI does not run the JSON deserialization step that `Json[...]` expects. The raw string hits the Pydantic validator, which rejects it because it expects a dict, not a string.

**Prevention:** Accept the field as a plain string and parse it manually.

```python
import json
from pydantic import BaseModel, field_validator

# Bad — Json type silently fails with form data
@app.post("/config")
async def update_config(
    settings: Json[dict] = Form(...),  # 422 error
):
    ...

# Good — manual parsing with clear error handling
@app.post("/config")
async def update_config(
    settings: str = Form(..., description="JSON string of settings"),
):
    try:
        parsed = json.loads(settings)
    except json.JSONDecodeError:
        raise HTTPException(status_code=422, detail="settings must be valid JSON")
    ...

# Also good — Pydantic model with validator
class ConfigForm(BaseModel):
    settings_raw: str

    @field_validator("settings_raw")
    @classmethod
    def parse_settings(cls, v: str) -> str:
        json.loads(v)  # validate, raises ValueError on bad JSON
        return v
```

**Detection:** Find `Json[` annotations in files that also use `Form`.

```bash
grep -rln "Json\[" app/ | xargs grep -l "Form("
```

---

## 5. ForwardRef OpenAPI Breakage

**Symptom:** Accessing `/docs` or `/openapi.json` produces a 500 Internal Server Error. The traceback references `PydanticSchemaGenerationError` or `NameError` mentioning an unresolved forward reference.

**Root Cause:** When a Pydantic model references another model by string name (a forward reference), Python defers resolution of that type. If the referenced model is defined later in the file or in a not-yet-imported module, the schema generator fails because it cannot resolve the string to an actual class at schema build time. This frequently happens with recursive or mutually-referential models.

**Prevention:** Call `model_rebuild()` after all referenced models are defined. Alternatively, structure imports so all models are available before the FastAPI app object processes them.

```python
from pydantic import BaseModel

class Comment(BaseModel):
    text: str
    replies: list["Comment"] = []  # forward ref to self

# Required — resolves the forward reference
Comment.model_rebuild()

# For cross-model references
class Author(BaseModel):
    name: str
    posts: list["Post"] = []

class Post(BaseModel):
    title: str
    author: "Author"

# Rebuild both after all are defined
Author.model_rebuild()
Post.model_rebuild()
```

**Detection:** Search for string-annotated type references in model definitions.

```bash
grep -rn 'list\["' app/models/
grep -rn '": "' app/models/ | grep -i "basemodel"
```

---

## 6. Pydantic v2 Union Type Changes

**Symptom:** Data that validated correctly under Pydantic v1 is now either rejected or matched to the wrong variant in a `Union` type. For example, `Union[DetailedItem, SimpleItem]` might always match `SimpleItem` even when `DetailedItem` is the better fit.

**Root Cause:** Pydantic v1 used a "best match" strategy for union validation — it tried all variants and picked the one that consumed the most fields. Pydantic v2 uses strict left-to-right evaluation — it accepts the first variant that validates without error. If a less-specific type appears first and happens to validate (because extra fields are ignored by default), it wins even when a more-specific type would be a better match.

**Prevention:** Order union variants from most-specific (most required fields) to least-specific. Alternatively, use discriminated unions with a literal tag field to make matching explicit.

```python
# Bad — SimpleItem matches first, DetailedItem never reached
from typing import Union

class SimpleItem(BaseModel):
    name: str

class DetailedItem(BaseModel):
    name: str
    description: str
    price: float

Item = Union[SimpleItem, DetailedItem]  # SimpleItem always wins

# Good — most specific first
Item = Union[DetailedItem, SimpleItem]

# Better — discriminated union, no ambiguity
from typing import Literal

class SimpleItem(BaseModel):
    kind: Literal["simple"]
    name: str

class DetailedItem(BaseModel):
    kind: Literal["detailed"]
    name: str
    description: str
    price: float

Item = Annotated[Union[SimpleItem, DetailedItem], Field(discriminator="kind")]
```

**Detection:** Find union type annotations in model files.

```bash
grep -rn "Union\[" app/models/ app/schemas/
```

---

## 7. ValueError in Validators Returns 500

**Symptom:** Sending invalid input to an endpoint returns a 500 Internal Server Error instead of a 422 Validation Error. The server logs show an unhandled `ValueError`.

**Root Cause:** Pydantic catches `ValueError` and `AssertionError` inside `@field_validator` and `@model_validator` decorated methods and converts them to validation errors (422). However, if a plain utility function raises `ValueError` during request processing — even if called from a validator — and that call happens outside the Pydantic validation context (for example, in a dependency or middleware), FastAPI does not catch it. The exception propagates as a 500.

**Prevention:** Confine `ValueError` raises to inside Pydantic validator methods. In utility functions, return sentinel values or raise custom exceptions that your error handler catches.

```python
# Bad — ValueError in utility function causes 500
def parse_date_range(start: str, end: str) -> tuple:
    if start > end:
        raise ValueError("start must be before end")  # 500 if called outside validator
    return (start, end)

@app.get("/report")
async def report(start: str, end: str):
    dates = parse_date_range(start, end)  # unhandled ValueError = 500
    ...

# Good — use HTTPException or custom exception in non-validator code
class InvalidDateRange(Exception):
    pass

def parse_date_range(start: str, end: str) -> tuple:
    if start > end:
        raise InvalidDateRange("start must be before end")
    return (start, end)

@app.exception_handler(InvalidDateRange)
async def handle_invalid_range(request, exc):
    return JSONResponse(status_code=422, content={"detail": str(exc)})

@app.get("/report")
async def report(start: str, end: str):
    dates = parse_date_range(start, end)  # now returns 422
    ...
```

**Detection:** Find `ValueError` raises in non-validator functions.

```bash
grep -rn "raise ValueError" app/ | grep -v "field_validator\|model_validator"
```

---

## 8. Blocking Calls in Async Code

**Symptom:** The entire server becomes unresponsive under load. Individual requests take seconds or minutes to complete even for simple operations. Health checks time out. CPU usage stays low while request latency spikes.

**Root Cause:** Python's `asyncio` event loop runs in a single thread. When an `async def` endpoint calls a blocking function — `requests.get()`, `time.sleep()`, a synchronous database driver — it blocks that thread. No other requests can be processed until the blocking call returns. With synchronous `def` endpoints, FastAPI runs them in a threadpool automatically, but `async def` endpoints run directly on the event loop.

**Prevention:** Use async-native libraries in `async def` endpoints. For unavoidable synchronous code, use `run_in_executor` or define the endpoint as `def` instead of `async def`.

```python
# Bad — blocks the entire event loop
import requests
import time

@app.get("/external")
async def fetch_external():
    resp = requests.get("https://api.example.com/data")  # blocks
    time.sleep(1)  # blocks
    return resp.json()

# Good — async alternatives
import httpx
import asyncio

@app.get("/external")
async def fetch_external():
    async with httpx.AsyncClient() as client:
        resp = await client.get("https://api.example.com/data")
    await asyncio.sleep(1)
    return resp.json()

# Acceptable — sync endpoint runs in threadpool automatically
@app.get("/external")
def fetch_external():  # no async — FastAPI uses threadpool
    resp = requests.get("https://api.example.com/data")
    return resp.json()
```

**Detection:** Find synchronous blocking calls in files with async endpoints.

```bash
grep -rn "import requests\|time\.sleep\|psycopg2\|sqlite3" app/
grep -rn "requests\.get\|requests\.post" app/ | xargs -I{} grep -l "async def" {}
```

---

## 9. Missing Await on Coroutines

**Symptom:** An async function returns `<coroutine object function_name at 0x...>` instead of its actual result. The operation the function performs (database write, HTTP call, file write) never executes. Python may emit a `RuntimeWarning: coroutine was never awaited` warning, but this warning is easy to miss in production logs.

**Root Cause:** Calling an `async def` function without `await` does not execute the function body. It creates a coroutine object and returns it immediately. The coroutine is never scheduled on the event loop, so its side effects never happen. This is a language-level behavior, not a FastAPI issue, but it surfaces frequently in FastAPI applications because of the mix of sync and async code.

**Prevention:** Always `await` calls to async functions. Enable the `RUF006` ruff rule to catch unawaited coroutines at lint time. Use `mypy` with `warn_unused_coroutines` or `pyright` for static analysis.

```python
# Bad — coroutine never awaited, user never created
async def create_user(name: str) -> User:
    user = User(name=name)
    await db.save(user)
    return user

@app.post("/users")
async def register(name: str):
    user = create_user(name)  # returns coroutine object, not User
    return {"id": user.id}   # AttributeError or wrong value

# Good — awaited correctly
@app.post("/users")
async def register(name: str):
    user = await create_user(name)  # executes and returns User
    return {"id": user.id}
```

Configure ruff to catch this:

```toml
# ruff.toml or pyproject.toml [tool.ruff]
[lint]
select = ["RUF006"]
```

**Detection:** Check for runtime warnings and lint violations.

```bash
grep -rn "RuntimeWarning.*coroutine" logs/
ruff check app/ --select RUF006
```

---

## 10. CORS Misconfiguration

**Symptom:** Browser console shows "Access to fetch at ... from origin ... has been blocked by CORS policy." Preflight `OPTIONS` requests return 403 or lack required headers. The API works from server-side code and curl but fails from browser JavaScript.

**Root Cause:** The `CORSMiddleware` configuration does not match the actual frontend origin. Common mistakes: omitting the port number (`http://localhost` vs `http://localhost:3000`), wrong protocol (`http` vs `https`), trailing slashes, or using `allow_origins=["*"]` while also sending credentials (which browsers reject per the CORS specification). Missing `allow_methods` or `allow_headers` blocks specific request types.

**Prevention:** Specify exact origins including protocol and port. Never use `["*"]` in production. Test CORS with an actual `OPTIONS` preflight request during development.

```python
# Bad — wildcard in production, missing specific origins
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],  # insecure and breaks credentials
    allow_methods=["GET", "POST"],  # forgot PUT, DELETE, PATCH
)

# Bad — missing port number
app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost"],  # frontend is on :3000
)

# Good — explicit origins, full method list
from app.config import settings

app.add_middleware(
    CORSMiddleware,
    allow_origins=[
        "http://localhost:3000",
        "https://app.example.com",
    ],
    allow_credentials=True,
    allow_methods=["GET", "POST", "PUT", "PATCH", "DELETE", "OPTIONS"],
    allow_headers=["Authorization", "Content-Type"],
)
```

Test CORS during development:

```bash
curl -X OPTIONS http://localhost:8000/api/resource \
  -H "Origin: http://localhost:3000" \
  -H "Access-Control-Request-Method: POST" \
  -H "Access-Control-Request-Headers: Authorization" \
  -v 2>&1 | grep -i "access-control"
```

**Detection:** Find wildcard origins and incomplete CORS configs.

```bash
grep -rn 'allow_origins.*\*' app/
grep -rn "CORSMiddleware" app/ -A 5
```
