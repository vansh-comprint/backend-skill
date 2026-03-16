# Exception Handling Reference

Consistent, frontend-friendly exception handling for FastAPI. Every error response follows the standard format: `{success, code, message, data, errors}`.

---

## 1. BaseAPIException

The base class for all application-level exceptions. It extends `HTTPException` and adds structured fields that the global handler uses to build the standard response.

```python
# core/exceptions.py
from typing import Any
from fastapi import HTTPException


class BaseAPIException(HTTPException):
    def __init__(
        self,
        status_code: int,
        message: str,
        data: Any = None,
        headers: dict[str, str] | None = None,
    ):
        self.message = message
        self.data = data
        super().__init__(status_code=status_code, detail=message, headers=headers)
```

**Fields:**

| Field | Type | Purpose |
|-------|------|---------|
| `status_code` | `int` | HTTP status code (400, 401, 403, 404, 409, 429) |
| `message` | `str` | Human-readable error description sent to the frontend |
| `data` | `Any` | Optional payload for additional context (defaults to `None`) |
| `headers` | `dict` | Optional HTTP headers (e.g., `Retry-After` for rate limiting) |

All concrete exception classes inherit from `BaseAPIException`. The global exception handler catches `BaseAPIException` and formats it into the standard response shape. This means services never need to construct `JSONResponse` objects directly — they raise an exception, and the handler does the rest.

---

## 2. Concrete Exception Types

Each exception type maps to a specific HTTP status code and provides a sensible default message.

```python
class NotFoundError(BaseAPIException):
    def __init__(self, resource: str, identifier: Any):
        super().__init__(
            status_code=404,
            message=f"{resource} with id '{identifier}' not found",
        )


class UnauthorizedError(BaseAPIException):
    def __init__(self, message: str = "Authentication required"):
        super().__init__(status_code=401, message=message)


class ForbiddenError(BaseAPIException):
    def __init__(self, message: str = "Insufficient permissions"):
        super().__init__(status_code=403, message=message)


class ConflictError(BaseAPIException):
    def __init__(self, resource: str, identifier: Any):
        super().__init__(
            status_code=409,
            message=f"{resource} '{identifier}' already exists",
        )


class BadRequestError(BaseAPIException):
    def __init__(self, message: str = "Bad request"):
        super().__init__(status_code=400, message=message)


class RateLimitError(BaseAPIException):
    def __init__(self):
        super().__init__(
            status_code=429,
            message="Rate limit exceeded. Try again later.",
            headers={"Retry-After": "60"},
        )
```

**Complete mapping:**

| Exception | HTTP Code | Default Message |
|-----------|-----------|-----------------|
| `NotFoundError` | 404 | `"{resource} with id '{id}' not found"` |
| `UnauthorizedError` | 401 | `"Authentication required"` |
| `ForbiddenError` | 403 | `"Insufficient permissions"` |
| `ConflictError` | 409 | `"{resource} '{id}' already exists"` |
| `BadRequestError` | 400 | `"Bad request"` |
| `RateLimitError` | 429 | `"Rate limit exceeded. Try again later."` |

`NotFoundError` and `ConflictError` accept `resource` and `identifier` parameters to build descriptive messages. `UnauthorizedError`, `ForbiddenError`, and `BadRequestError` accept a custom message string. `RateLimitError` takes no arguments and always includes a `Retry-After` header.

---

## 3. Global Exception Handlers

Three handlers catch every error type and produce the consistent response format. They are registered on the FastAPI app instance during startup.

```python
# core/exceptions.py (continued)
import logging
from fastapi import FastAPI, Request
from fastapi.responses import JSONResponse
from fastapi.exceptions import RequestValidationError

logger = logging.getLogger("app.exceptions")


def register_exception_handlers(app: FastAPI):

    @app.exception_handler(BaseAPIException)
    async def api_exception_handler(request: Request, exc: BaseAPIException):
        request_id = getattr(request.state, "request_id", "unknown")

        logger.warning(
            "API error: %s [%d]",
            exc.message,
            exc.status_code,
            extra={"request_id": request_id, "path": request.url.path},
        )

        return JSONResponse(
            status_code=exc.status_code,
            content={
                "success": False,
                "code": exc.status_code,
                "message": exc.message,
                "data": exc.data,
                "errors": None,
            },
            headers={"X-Request-ID": request_id},
        )

    @app.exception_handler(RequestValidationError)
    async def validation_exception_handler(
        request: Request, exc: RequestValidationError
    ):
        request_id = getattr(request.state, "request_id", "unknown")

        errors = []
        for error in exc.errors():
            loc_parts = error["loc"]
            field_parts = [
                str(p) for p in loc_parts
                if p not in ("body", "query", "path", "header")
            ]
            field = ".".join(field_parts) if field_parts else str(loc_parts[-1])

            errors.append({
                "field": field,
                "message": error["msg"],
            })

        logger.warning(
            "Validation error on %s",
            request.url.path,
            extra={"request_id": request_id, "errors": errors},
        )

        return JSONResponse(
            status_code=422,
            content={
                "success": False,
                "code": 422,
                "message": "Validation failed",
                "data": None,
                "errors": errors,
            },
            headers={"X-Request-ID": request_id},
        )

    @app.exception_handler(Exception)
    async def unhandled_exception_handler(request: Request, exc: Exception):
        request_id = getattr(request.state, "request_id", "unknown")

        logger.error(
            "Unhandled exception: %s",
            str(exc),
            exc_info=True,
            extra={"request_id": request_id, "path": request.url.path},
        )

        return JSONResponse(
            status_code=500,
            content={
                "success": False,
                "code": 500,
                "message": "An unexpected error occurred",
                "data": None,
                "errors": None,
            },
            headers={"X-Request-ID": request_id},
        )
```

### Handler Behavior Summary

**BaseAPIException handler** — Catches all custom domain exceptions. Logs at WARNING level with the request ID and URL path. Returns the exception's status code, message, and optional data. Sets `errors: None` since these are not validation errors.

**RequestValidationError handler** — Catches Pydantic validation failures (422). Transforms the raw Pydantic error list into a clean `[{field, message}]` array. Strips location prefixes (`body`, `query`, `path`, `header`) from field names so they match frontend form field names directly. Logs at WARNING level.

**Unhandled Exception handler** — The catch-all for anything not covered above. Always returns HTTP 500 with a generic message. Logs the full stack trace at ERROR level with `exc_info=True`. The stack trace is never exposed to the frontend — only the generic "An unexpected error occurred" message is sent.

### Registration

Call `register_exception_handlers(app)` in your `main.py` during app initialization:

```python
from fastapi import FastAPI
from app.core.exceptions import register_exception_handlers

app = FastAPI()
register_exception_handlers(app)
```

---

## 4. Validation Error Formatting

Pydantic's default validation errors include location tuples that leak internal details to the frontend. The validation handler transforms these into a clean, consistent format.

### Transformation Rules

| Pydantic Location | Cleaned Field Name |
|---|---|
| `("body", "email")` | `"email"` |
| `("body", "address", "city")` | `"address.city"` |
| `("query", "page")` | `"page"` |
| `("path", "user_id")` | `"user_id"` |
| `("header", "x-token")` | `"x-token"` |

The prefixes `body`, `query`, `path`, and `header` are stripped. Remaining parts are joined with dots. This means field names in the error response match the field names the frontend form uses, enabling direct field-level error display without string manipulation on the client side.

### Example Response

A request with invalid `email` and `password` fields produces:

```json
{
    "success": false,
    "code": 422,
    "message": "Validation failed",
    "data": null,
    "errors": [
        {"field": "email", "message": "Invalid email format"},
        {"field": "password", "message": "Must be at least 8 characters"}
    ]
}
```

For nested models, the dot notation preserves the hierarchy:

```json
{
    "success": false,
    "code": 422,
    "message": "Validation failed",
    "data": null,
    "errors": [
        {"field": "address.city", "message": "Field required"},
        {"field": "address.zip_code", "message": "String should have at most 10 characters"}
    ]
}
```

---

## 5. Service to HTTP Exception Mapping

Services raise domain exceptions, not `HTTPException` directly. This keeps the service layer decoupled from HTTP concerns. The global exception handlers perform the mapping to HTTP responses automatically.

### Pattern

```python
from app.core.exceptions import NotFoundError, ConflictError


class UserService:
    def __init__(self, repository: UserRepositoryProtocol):
        self.repository = repository

    async def get_by_id(self, user_id: int) -> User:
        user = await self.repository.get_by_id(user_id)
        if not user:
            raise NotFoundError("User", user_id)
        return user

    async def create_user(self, data: UserCreate) -> User:
        existing = await self.repository.get_by_email(data.email)
        if existing:
            raise ConflictError("User", data.email)
        return await self.repository.create(data.model_dump())

    async def deactivate_user(self, user_id: int, requesting_user: User) -> User:
        user = await self.repository.get_by_id(user_id)
        if not user:
            raise NotFoundError("User", user_id)
        if requesting_user.role != "admin":
            raise ForbiddenError("Only admins can deactivate users")
        return await self.repository.update(user, {"is_active": False})
```

### Why Not Raise HTTPException Directly?

1. **Decoupling** — Services should not know about HTTP. They express domain rules ("this user does not exist") and the framework layer translates to HTTP ("404 Not Found").
2. **Testability** — Unit tests can catch `NotFoundError` without importing FastAPI or dealing with HTTP response objects.
3. **Consistency** — The global handler guarantees every error response follows the standard format. If services build their own `JSONResponse` objects, format consistency depends on every developer remembering the shape.
4. **Logging** — The global handler logs every error with request context (request ID, path, method). Services raising exceptions get this logging for free.

### Complete Flow

```
Client Request
    -> FastAPI Route Handler
        -> Service Method
            -> raises NotFoundError("User", 42)
        <- exception propagates
    <- Global Handler catches BaseAPIException
    <- Returns JSONResponse {success: false, code: 404, message: "User with id '42' not found", data: null, errors: null}
```

---

## 6. Error Logging with Request Context

Every exception handler logs with structured context fields. This enables filtering and correlation in log aggregation systems.

### Logged Fields

| Field | Source | Purpose |
|-------|--------|---------|
| `request_id` | `request.state.request_id` | Correlate all logs for a single request |
| `path` | `request.url.path` | Identify which endpoint failed |
| `exc.message` | Exception instance | Human-readable error description |
| `exc.status_code` | Exception instance | HTTP status code |
| `exc_info` | Python runtime | Full stack trace (unhandled exceptions only) |

### Log Levels by Handler

| Handler | Level | Rationale |
|---------|-------|-----------|
| `BaseAPIException` | `WARNING` | Expected application errors (404, 401, 409) |
| `RequestValidationError` | `WARNING` | Client-sent invalid data |
| Unhandled `Exception` | `ERROR` | Unexpected failures requiring investigation |

### Request ID Middleware

The `request_id` is expected to be set by middleware before the exception handlers run. A typical implementation:

```python
import uuid
from fastapi import Request


async def request_id_middleware(request: Request, call_next):
    request_id = request.headers.get("X-Request-ID", str(uuid.uuid4()))
    request.state.request_id = request_id
    response = await call_next(request)
    response.headers["X-Request-ID"] = request_id
    return response
```

This middleware either uses an incoming `X-Request-ID` header (for distributed tracing) or generates a new UUID. The request ID is then available in `request.state.request_id` for all exception handlers.

### Structured Logging Example

With a JSON log formatter, the logged output for a `NotFoundError` looks like:

```json
{
    "level": "WARNING",
    "logger": "app.exceptions",
    "message": "API error: User with id '42' not found [404]",
    "request_id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "path": "/api/v1/users/42",
    "timestamp": "2024-01-15T09:30:45.123Z"
}
```

For unhandled exceptions, `exc_info=True` ensures the full traceback is captured in the log record. This traceback is logged server-side only and never included in the HTTP response sent to the frontend. The client receives only:

```json
{
    "success": false,
    "code": 500,
    "message": "An unexpected error occurred",
    "data": null,
    "errors": null
}
```

### Extending with User Context

For authenticated endpoints, include the user ID in log context by extending the exception handlers:

```python
@app.exception_handler(BaseAPIException)
async def api_exception_handler(request: Request, exc: BaseAPIException):
    request_id = getattr(request.state, "request_id", "unknown")
    user_id = getattr(request.state, "user_id", None)

    logger.warning(
        "API error: %s [%d]",
        exc.message,
        exc.status_code,
        extra={
            "request_id": request_id,
            "path": request.url.path,
            "method": request.method,
            "user_id": user_id,
        },
    )

    return JSONResponse(
        status_code=exc.status_code,
        content={
            "success": False,
            "code": exc.status_code,
            "message": exc.message,
            "data": exc.data,
            "errors": None,
        },
        headers={"X-Request-ID": request_id},
    )
```

Adding `method` and `user_id` to the `extra` dict provides full request context for debugging. The authentication middleware should set `request.state.user_id` after token validation, making it available to all downstream handlers.
