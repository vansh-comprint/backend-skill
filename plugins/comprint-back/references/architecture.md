# Architecture Reference

## 4-Layer Architecture

All FastAPI projects follow a strict 4-layer dependency chain. Dependencies flow downward only — never skip layers.

```
Endpoints -> Services -> Repositories -> Models
```

### Endpoints (API Layer)

- Accept HTTP requests, validate input via Pydantic schemas
- Call exactly one service method per endpoint
- Return `StandardResponse` or `PaginatedResponse`
- No business logic, no database access, no direct model usage

```python
@router.post("/", response_model=StandardResponse[UserResponse], status_code=201)
async def create_user(data: UserCreate, service: UserServiceDep):
    user = await service.create_user(data)
    return StandardResponse(code=201, data=user, message="User created successfully")
```

### Services (Business Logic Layer)

- Contain all business rules, validation logic, orchestration
- Depend on repository Protocols, never concrete implementations
- Raise domain-specific exceptions (not HTTP exceptions)
- No direct database queries, no HTTP awareness

```python
class UserService:
    def __init__(self, repo: UserRepositoryProtocol):
        self.repo = repo

    async def create_user(self, data: UserCreate) -> User:
        existing = await self.repo.get_by_email(data.email)
        if existing:
            raise DuplicateEntityError("User with this email already exists")
        return await self.repo.create(data)
```

### Repositories (Data Access Layer)

- Pure data access — SQL queries, cache reads/writes
- Extend `BaseAsyncRepository` for generic CRUD
- Implement a Protocol interface per repository
- No business logic, no HTTP concepts

```python
from typing import Protocol

class UserRepositoryProtocol(Protocol):
    async def get_by_id(self, user_id: int) -> User | None: ...
    async def get_by_email(self, email: str) -> User | None: ...
    async def create(self, data: UserCreate) -> User: ...

class UserRepository:
    def __init__(self, db: AsyncSession):
        self.db = db

    async def get_by_id(self, user_id: int) -> User | None:
        result = await self.db.execute(select(User).where(User.id == user_id))
        return result.scalar_one_or_none()
```

### Models (ORM Layer)

- SQLAlchemy ORM class definitions only
- Columns, relationships, mixins (Timestamp, SoftDelete)
- No logic beyond declarative mapping

```python
from sqlalchemy import String, Boolean
from sqlalchemy.orm import Mapped, mapped_column

from app.models.base import Base, TimestampMixin, SoftDeleteMixin


class User(Base, TimestampMixin, SoftDeleteMixin):
    __tablename__ = "users"

    id: Mapped[int] = mapped_column(primary_key=True)
    email: Mapped[str] = mapped_column(String(255), unique=True, index=True)
    full_name: Mapped[str] = mapped_column(String(100))
    hashed_password: Mapped[str] = mapped_column(String(255))
    is_active: Mapped[bool] = mapped_column(Boolean, default=True)
```

### Layer Violation Quick Reference

| Violation | Example | Fix |
|-----------|---------|-----|
| Endpoint calls repo directly | `repo.get_by_id()` in router | Route through service |
| Service imports FastAPI | `from fastapi import HTTPException` | Raise domain exception, catch in handler |
| Repository contains business logic | `if user.is_admin:` in repo | Move condition to service |
| Model contains query logic | `@classmethod def find_active()` | Move to repository |

---

## When to Use Simple vs Modular

| Criteria | Simple | Modular |
|----------|--------|---------|
| Endpoints | < 50 | 50+ |
| Developers | 1-4 | 4+ |
| Domains | Single | Multiple (billing, users, analytics) |
| Focus | MVP, prototype, fast delivery | Enterprise, long-term maintenance |
| Structure | Flat directories | Domain-based modules |

Start with simple. Migrate to modular when merge conflicts increase, distinct domains emerge, or endpoint count crosses 50.

### Migration Path (Simple to Modular)

When the project outgrows the simple structure:

1. Create `app/modules/` and `app/shared/` directories
2. Move each domain's models, schemas, services, and repositories into its own module
3. Move shared code (base classes, common schemas, utilities) into `shared/`
4. Add a `protocols.py` to each module for its repository interfaces
5. Create module-level `dependencies.py` with Annotated DI aliases
6. Update the API router aggregation to import from modules
7. Mirror the test directory structure to match modules
8. Run the full test suite to verify nothing broke

---

## Simple Project Structure

Flat directory layout — one folder per layer, all domains mixed together.

```
project_root/
├── app/
│   ├── __init__.py
│   ├── main.py                     # FastAPI app + lifespan
│   ├── config/
│   │   ├── __init__.py
│   │   ├── settings.py             # Pydantic BaseSettings + SettingsConfigDict
│   │   └── database.py             # Async SQLAlchemy engine + session
│   ├── api/
│   │   ├── __init__.py
│   │   ├── dependencies.py         # Annotated type aliases for DI
│   │   └── v1/
│   │       ├── __init__.py
│   │       ├── router.py           # Aggregates all v1 endpoint routers
│   │       └── endpoints/
│   │           ├── __init__.py
│   │           ├── auth.py
│   │           ├── users.py
│   │           ├── items.py
│   │           └── health.py
│   ├── core/
│   │   ├── __init__.py
│   │   ├── security.py             # JWT, password hashing
│   │   ├── exceptions.py           # Custom exceptions + handlers
│   │   └── middleware.py           # RequestID, security headers
│   ├── models/
│   │   ├── __init__.py
│   │   ├── base.py                 # Base, TimestampMixin, SoftDeleteMixin
│   │   ├── User.py
│   │   └── Item.py
│   ├── schemas/
│   │   ├── __init__.py
│   │   ├── common.py               # StandardResponse, PaginatedResponse
│   │   ├── UserSchema.py
│   │   └── ItemSchema.py
│   ├── services/
│   │   ├── __init__.py
│   │   ├── UserService.py
│   │   └── ItemService.py
│   ├── repositories/
│   │   ├── __init__.py
│   │   ├── protocols.py            # All repository Protocols
│   │   ├── base.py                 # BaseAsyncRepository
│   │   ├── UserRepository.py
│   │   └── ItemRepository.py
│   └── utils/
│       ├── __init__.py
│       ├── logger.py
│       └── helpers.py
├── tests/
│   ├── conftest.py
│   ├── unit/
│   └── integration/
├── migrations/
│   ├── env.py
│   └── versions/
├── pyproject.toml
├── Makefile
├── Dockerfile
└── docker-compose.yml
```

File naming: **class name = file name** in PascalCase. `User.py` contains `class User`. `UserService.py` contains `class UserService`. Endpoint files use lowercase (`users.py`).

Key characteristics of the simple layout:
- All models in one `models/` directory, all services in one `services/` directory
- Single `protocols.py` holds every repository Protocol
- Single `dependencies.py` holds every Annotated DI alias
- Router aggregation in `api/v1/router.py` imports from `endpoints/`
- Tests split by type (unit vs integration), not by domain

---

## Modular Project Structure

Domain-based layout — each business domain is a self-contained module.

```
project_root/
├── app/
│   ├── __init__.py
│   ├── main.py
│   ├── config/
│   │   ├── settings.py
│   │   └── database.py
│   ├── core/
│   │   ├── security.py
│   │   ├── exceptions.py
│   │   ├── middleware.py
│   │   └── dependencies.py        # Global DI: DbSession, CurrentUser
│   ├── shared/
│   │   ├── schemas/
│   │   │   └── common.py          # StandardResponse, PaginatedResponse
│   │   ├── models/
│   │   │   └── base.py            # Base, TimestampMixin, SoftDeleteMixin
│   │   ├── repositories/
│   │   │   ├── protocols.py       # Base repository Protocol
│   │   │   └── base.py            # BaseAsyncRepository
│   │   ├── services/
│   │   │   └── base.py            # BaseService
│   │   ├── events/
│   │   │   └── event_bus.py       # Async event bus
│   │   └── utils/
│   ├── modules/
│   │   ├── auth/
│   │   ├── users/
│   │   ├── billing/
│   │   └── notifications/
│   └── api/
│       └── v1/
│           └── router.py          # Aggregates all module routers
├── tests/
│   └── modules/
│       ├── auth/
│       └── users/
├── migrations/
├── pyproject.toml
└── Dockerfile
```

### Module Anatomy

Every module is self-contained with its own 4-layer stack:

```
module_name/
├── __init__.py          # Exports router only
├── api/endpoints.py     # FastAPI router
├── services/            # Business logic
├── repositories/
│   ├── protocols.py     # Module-specific Protocol
│   └── EntityRepo.py    # Concrete implementation
├── models/              # SQLAlchemy models
├── schemas/             # Pydantic schemas
├── dependencies.py      # Module Annotated DI aliases
└── events/              # Optional event handlers
```

Module `__init__.py` exports only the router:

```python
from app.modules.users.api.endpoints import router as users_router

__all__ = ["users_router"]
```

Router aggregation wires modules together:

```python
# app/api/v1/router.py
from fastapi import APIRouter
from app.modules.auth import auth_router
from app.modules.users import users_router
from app.modules.billing import billing_router

api_v1_router = APIRouter()
api_v1_router.include_router(auth_router, prefix="/auth", tags=["Auth"])
api_v1_router.include_router(users_router, prefix="/users", tags=["Users"])
api_v1_router.include_router(billing_router, prefix="/billing", tags=["Billing"])
```

---

## Dependency Injection Patterns

All DI uses `Annotated` type aliases defined in a central `dependencies.py` file.

### Database Session

```python
from typing import Annotated
from fastapi import Depends
from sqlalchemy.ext.asyncio import AsyncSession

DbSession = Annotated[AsyncSession, Depends(get_db)]
```

### Service Injection

Wire repository into service via a factory function:

```python
def get_user_service(db: DbSession) -> UserService:
    repo = UserRepository(db)
    return UserService(repo)

UserServiceDep = Annotated[UserService, Depends(get_user_service)]
```

### Auth / Current User

```python
CurrentUser = Annotated[User, Depends(get_current_user_from_token)]
```

### Usage in Endpoints

Endpoints receive fully wired dependencies through clean type annotations:

```python
@router.get("/me", response_model=StandardResponse[UserResponse])
async def get_me(user: CurrentUser, service: UserServiceDep):
    profile = await service.get_profile(user.id)
    return StandardResponse(data=profile)
```

In **simple** projects, all aliases live in `app/api/dependencies.py`. In **modular** projects, global aliases (`DbSession`, `CurrentUser`) live in `app/core/dependencies.py`, and module-specific aliases (`UserServiceDep`) live in each module's `dependencies.py`.

### Swapping Implementations

The factory function pattern makes it trivial to swap repository implementations without changing any endpoint or service code:

```python
# Switch from direct DB to cached repository
def get_user_service(db: DbSession) -> UserService:
    base_repo = UserRepository(db)
    cached_repo = CachedUserRepository(base_repo, redis_client)
    return UserService(cached_repo)  # Service unchanged — depends on Protocol
```

---

## Inter-Module Communication

Modules never import from each other's internals. Three patterns for cross-module interaction, ordered by preference.

### 1. Event Bus (Side Effects)

Best for fire-and-forget actions: sending emails, logging audit trails, syncing caches.

```python
# shared/events/event_bus.py
from typing import Any, Callable

class EventBus:
    _handlers: dict[str, list[Callable]] = {}

    @classmethod
    def subscribe(cls, event_name: str, handler: Callable):
        cls._handlers.setdefault(event_name, []).append(handler)

    @classmethod
    async def publish(cls, event_name: str, data: Any):
        for handler in cls._handlers.get(event_name, []):
            await handler(data)
```

Publisher (users module):
```python
await EventBus.publish("user.created", {"user_id": user.id, "email": user.email})
```

Subscriber (notifications module):
```python
EventBus.subscribe("user.created", send_welcome_email)
```

### 2. Shared Protocol Interface

When one module needs to read data owned by another, define a Protocol in `shared/`:

```python
# shared/repositories/protocols.py
from typing import Protocol

class UserLookupProtocol(Protocol):
    async def get_user_email(self, user_id: int) -> str | None: ...
```

The billing module depends on the Protocol, never on the users module directly.

### 3. API-Level Composition

For read-only cross-module data in a single endpoint, inject multiple services:

```python
@router.get("/dashboard")
async def dashboard(user_svc: UserServiceDep, billing_svc: BillingServiceDep):
    user = await user_svc.get_current()
    invoices = await billing_svc.get_user_invoices(user.id)
    return StandardResponse(data={"user": user, "invoices": invoices})
```
