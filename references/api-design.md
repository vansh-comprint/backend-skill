# API Design Reference

## Standard Response Format

Every response -- success and error -- uses the same 5-field shape. This lets the frontend handle all API calls with a single response handler.

### The 5 Fields

| Field | Type | Description |
|-------|------|-------------|
| `success` | `bool` | `true` for 2xx, `false` for errors |
| `code` | `int` | HTTP status code |
| `message` | `str` | Human-readable description |
| `data` | `any` | Response payload (single object, list, or `null`) |
| `errors` | `list\|null` | Validation field errors or `null` |

### Success Response

```json
{
  "success": true,
  "code": 200,
  "message": "User retrieved successfully",
  "data": { "id": 1, "email": "alice@example.com" },
  "errors": null
}
```

### Validation Error (422)

```json
{
  "success": false,
  "code": 422,
  "message": "Validation failed",
  "data": null,
  "errors": [
    { "field": "email", "message": "Invalid email format" },
    { "field": "password", "message": "Must be at least 8 characters" }
  ]
}
```

### Non-Validation Error (401, 403, 404, 409, 500)

```json
{
  "success": false,
  "code": 404,
  "message": "User with id '99' not found",
  "data": null,
  "errors": null
}
```

### Pydantic Models

```python
from typing import Any, Generic, TypeVar
from pydantic import BaseModel

T = TypeVar("T")


class FieldError(BaseModel):
    field: str
    message: str


class StandardResponse(BaseModel, Generic[T]):
    success: bool = True
    code: int = 200
    message: str = "Success"
    data: T | None = None
    errors: list[FieldError] | None = None


class PaginatedResponse(BaseModel, Generic[T]):
    success: bool = True
    code: int = 200
    message: str = "Success"
    data: list[T] = []
    errors: list[FieldError] | None = None
    total: int = 0
    page: int = 1
    page_size: int = 20
    total_pages: int = 0
```

`StandardResponse` wraps single objects. `PaginatedResponse` wraps lists with pagination metadata. Both carry the same 5 base fields (`success`, `code`, `message`, `data`, `errors`).

### Exception Handler Integration

Map custom exceptions to the standard response format in a global handler:

```python
from fastapi import FastAPI, Request
from fastapi.responses import JSONResponse


class NotFoundError(Exception):
    def __init__(self, entity: str, entity_id: int | str):
        self.entity = entity
        self.entity_id = entity_id


class DuplicateEntityError(Exception):
    def __init__(self, message: str):
        self.message = message


def register_exception_handlers(app: FastAPI):
    @app.exception_handler(NotFoundError)
    async def not_found_handler(request: Request, exc: NotFoundError):
        return JSONResponse(
            status_code=404,
            content=StandardResponse(
                success=False,
                code=404,
                message=f"{exc.entity} with id '{exc.entity_id}' not found",
                data=None,
                errors=None,
            ).model_dump(),
        )

    @app.exception_handler(DuplicateEntityError)
    async def duplicate_handler(request: Request, exc: DuplicateEntityError):
        return JSONResponse(
            status_code=409,
            content=StandardResponse(
                success=False,
                code=409,
                message=exc.message,
                data=None,
                errors=None,
            ).model_dump(),
        )
```

Every error path produces the same 5 fields. The frontend never encounters a response shape it does not expect.

---

## Endpoint Patterns

### Create (POST)

```python
@router.post("/", response_model=StandardResponse[UserResponse], status_code=201)
async def create_user(data: UserCreate, service: UserServiceDep):
    user = await service.create_user(data)
    return StandardResponse(code=201, data=user, message="User created successfully")
```

- Status code 201 on success
- Request body validated by `UserCreate` schema
- Returns created resource in `data`

### Read One (GET)

```python
@router.get("/{user_id}", response_model=StandardResponse[UserResponse])
async def get_user(user_id: int, service: UserServiceDep):
    user = await service.get_by_id(user_id)
    return StandardResponse(data=user)
```

- Path parameter typed as `int` for automatic validation
- Service raises `NotFoundError` if missing (caught by exception handler)

### List with Pagination (GET)

```python
from fastapi import Query

@router.get("/", response_model=PaginatedResponse[UserResponse])
async def list_users(
    service: UserServiceDep,
    page: int = Query(1, ge=1),
    page_size: int = Query(20, ge=1, le=100),
):
    users, total = await service.get_paginated(page, page_size)
    return PaginatedResponse(
        data=users,
        total=total,
        page=page,
        page_size=page_size,
        total_pages=(total + page_size - 1) // page_size,
    )
```

- `page` starts at 1, `page_size` capped at 100
- Service returns `(items, total_count)` tuple
- `total_pages` calculated with ceiling division

### Update (PATCH)

```python
@router.patch("/{user_id}", response_model=StandardResponse[UserResponse])
async def update_user(user_id: int, data: UserUpdate, service: UserServiceDep):
    user = await service.update_user(user_id, data)
    return StandardResponse(data=user, message="User updated successfully")
```

- PATCH for partial updates (all fields optional in `UserUpdate`)
- Returns the updated resource

### Delete (DELETE)

```python
@router.delete("/{user_id}", response_model=StandardResponse[None])
async def delete_user(user_id: int, service: UserServiceDep):
    await service.delete_user(user_id)
    return StandardResponse(message="User deleted successfully")
```

- `data` defaults to `None` (nothing to return)
- Soft delete preferred at the repository layer

### Get Current User (GET /me)

```python
@router.get("/me", response_model=StandardResponse[UserResponse])
async def get_me(user: CurrentUser):
    return StandardResponse(data=user)
```

- Uses `CurrentUser` Annotated dependency for auth
- No service call needed -- user already resolved by dependency chain

### Search / Filter (GET with query params)

```python
@router.get("/search", response_model=PaginatedResponse[UserResponse])
async def search_users(
    service: UserServiceDep,
    q: str = Query("", max_length=100),
    is_active: bool | None = Query(None),
    page: int = Query(1, ge=1),
    page_size: int = Query(20, ge=1, le=100),
):
    users, total = await service.search(q=q, is_active=is_active, page=page, page_size=page_size)
    return PaginatedResponse(
        data=users,
        total=total,
        page=page,
        page_size=page_size,
        total_pages=(total + page_size - 1) // page_size,
    )
```

- Optional filters default to `None` (no filtering)
- Reuses the same `PaginatedResponse` wrapper

---

## Schema Design

### Base / Create / Update / Response Pattern

Every entity gets four schemas. Never reuse a single schema for multiple operations.

```python
from pydantic import BaseModel, EmailStr, Field, ConfigDict
from datetime import datetime


class UserBase(BaseModel):
    """Shared fields between create and response."""
    email: EmailStr
    full_name: str = Field(..., min_length=1, max_length=100)


class UserCreate(UserBase):
    """Fields required to create a user. Includes write-only fields."""
    password: str = Field(..., min_length=8, max_length=128)


class UserUpdate(BaseModel):
    """All fields optional for PATCH semantics. Does NOT inherit Base."""
    email: EmailStr | None = None
    full_name: str | None = Field(None, min_length=1, max_length=100)


class UserResponse(UserBase):
    """Fields returned to the client. Never includes secrets."""
    id: int
    is_active: bool
    created_at: datetime

    model_config = ConfigDict(from_attributes=True)
```

### Rules

1. **UserBase** -- shared readable fields, inherited by Create and Response
2. **UserCreate** inherits Base, adds write-only fields (`password`)
3. **UserUpdate** does NOT inherit Base -- all fields are `Optional` for PATCH
4. **UserResponse** inherits Base, adds server-generated fields (`id`, `created_at`)
5. Response schemas use `ConfigDict(from_attributes=True)` to convert ORM objects
6. Never expose passwords, internal IDs, or sensitive data in Response
7. Use `Field()` for validation constraints (`min_length`, `max_length`, `ge`, `le`)
8. Python 3.12+ syntax: `str | None` instead of `Optional[str]`

---

## Annotated Dependency Injection

Define all DI as `Annotated` type aliases in `dependencies.py`. Endpoints receive fully wired dependencies through clean parameter annotations.

### Type Alias Definitions

```python
from typing import Annotated
from fastapi import Depends
from sqlalchemy.ext.asyncio import AsyncSession

# Database session
DBSession = Annotated[AsyncSession, Depends(get_db)]

# Auth
CurrentUser = Annotated[User, Depends(get_current_user)]

# Services (factory wires repo -> service)
def get_user_service(db: DBSession) -> UserService:
    repo = UserRepository(db)
    return UserService(repo)

UserServiceDep = Annotated[UserService, Depends(get_user_service)]
ItemServiceDep = Annotated[ItemService, Depends(get_item_service)]
```

### Usage in Endpoints

```python
@router.get("/me", response_model=StandardResponse[UserResponse])
async def get_me(user: CurrentUser, service: UserServiceDep):
    profile = await service.get_profile(user.id)
    return StandardResponse(data=profile)
```

### Benefits

- Endpoint signatures stay clean and readable
- DI wiring is centralized, not scattered across endpoints
- Swapping implementations (e.g., `CachedUserRepository` for `UserRepository`) requires changing one factory function
- FastAPI resolves the full dependency chain automatically

### Factory Pattern for Multiple Implementations

```python
def get_user_service(db: DBSession) -> UserService:
    repo = UserRepository(db)
    return UserService(repo)

# Swap to cached version — no endpoint changes needed
def get_user_service(db: DBSession) -> UserService:
    base_repo = UserRepository(db)
    cached_repo = CachedUserRepository(base_repo, redis_client)
    return UserService(cached_repo)
```

---

## API Versioning

URL path approach: `/api/v1/`, `/api/v2/`.

```python
app.include_router(api_v1_router, prefix="/api/v1")
app.include_router(api_v2_router, prefix="/api/v2")
```

Create a new version only for **breaking changes**: removed fields, changed types, removed endpoints. Non-breaking changes (new optional fields, new endpoints) go into the current version.

Breaking changes requiring a new version:
- Removing a field from a response
- Changing a field's type (e.g., `string` to `int`)
- Removing an endpoint
- Changing required/optional status of request fields

Non-breaking changes (add to current version):
- Adding a new optional field to a response
- Adding a new endpoint
- Adding a new optional query parameter

### Deprecation Middleware

Add sunset headers to deprecated API versions:

```python
from starlette.middleware.base import BaseHTTPMiddleware

class APIDeprecationMiddleware(BaseHTTPMiddleware):
    DEPRECATED = {
        "/api/v1": {"sunset": "2026-06-01", "msg": "Migrate to /api/v2"},
    }

    async def dispatch(self, request, call_next):
        response = await call_next(request)
        for prefix, info in self.DEPRECATED.items():
            if request.url.path.startswith(prefix):
                response.headers["Deprecation"] = "true"
                response.headers["Sunset"] = info["sunset"]
                response.headers["X-Deprecation-Notice"] = info["msg"]
        return response
```

---

## TypeScript Frontend Contract

The standard response format maps directly to TypeScript interfaces. Frontend teams use these types for a single API handler.

### Response Types

```typescript
interface FieldError {
  field: string;
  message: string;
}

interface ApiResponse<T> {
  success: boolean;
  code: number;
  message: string;
  data: T | null;
  errors: FieldError[] | null;
}

interface PaginatedResponse<T> {
  success: boolean;
  code: number;
  message: string;
  data: T[];
  errors: FieldError[] | null;
  total: number;
  page: number;
  page_size: number;
  total_pages: number;
}
```

### Generic API Client

```typescript
async function apiCall<T>(url: string, options?: RequestInit): Promise<ApiResponse<T>> {
  const response = await fetch(url, {
    headers: { "Content-Type": "application/json" },
    ...options,
  });
  const result: ApiResponse<T> = await response.json();

  if (!result.success) {
    if (result.errors) {
      // Validation errors -- map to form fields
      throw new ValidationError(result.errors);
    }
    // General error -- show message to user
    throw new ApiError(result.code, result.message);
  }

  return result;
}
```

### Usage Example

```typescript
// Single resource
const { data: user } = await apiCall<User>("/api/v1/users/1");

// Paginated list
const result = await apiCall<PaginatedResponse<User>>("/api/v1/users?page=1&page_size=20");
console.log(result.data); // User[]
console.log(result.total_pages);

// Create with error handling
try {
  const { data: newUser } = await apiCall<User>("/api/v1/users", {
    method: "POST",
    body: JSON.stringify({ email: "alice@example.com", full_name: "Alice", password: "secret123" }),
  });
} catch (err) {
  if (err instanceof ValidationError) {
    // err.errors = [{ field: "email", message: "Already exists" }]
    setFormErrors(err.errors);
  }
}
```
