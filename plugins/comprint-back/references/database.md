# Database Reference

Async database configuration, repository patterns, migration management, and query optimization for FastAPI with PostgreSQL and AsyncPG.

---

## 1. Async Engine & Session Setup

PostgreSQL with AsyncPG provides the async database driver. The engine and session factory are configured once at module level, then injected via FastAPI's dependency system.

### Connection String Format

```
postgresql+asyncpg://user:password@host:5432/dbname
```

The `+asyncpg` dialect is required. Standard `postgresql://` will not work with async SQLAlchemy.

### Engine and Session Factory

```python
# config/database.py
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession, async_sessionmaker
from app.config.settings import settings

engine = create_async_engine(
    settings.DATABASE_URL,          # postgresql+asyncpg://user:pass@host/db
    pool_size=settings.DATABASE_POOL_SIZE,
    max_overflow=settings.DATABASE_MAX_OVERFLOW,
    pool_timeout=30,
    pool_recycle=1800,
    pool_pre_ping=True,
    echo=settings.DEBUG,
)

AsyncSessionLocal = async_sessionmaker(
    bind=engine,
    class_=AsyncSession,
    expire_on_commit=False,
)
```

**Pool configuration:**

| Parameter | Purpose |
|-----------|---------|
| `pool_size` | Base number of persistent connections |
| `max_overflow` | Extra connections allowed above `pool_size` |
| `pool_timeout` | Seconds to wait for a connection from the pool before raising |
| `pool_recycle` | Seconds before a connection is recycled (set to 1800 to avoid stale connections) |
| `pool_pre_ping` | Issues a lightweight `SELECT 1` before handing out a connection to verify liveness |

Setting `expire_on_commit=False` prevents lazy-load attribute access errors after commit in async contexts.

### Dependency Injection

```python
async def get_db() -> AsyncSession:
    async with AsyncSessionLocal() as session:
        try:
            yield session
            await session.commit()
        except Exception:
            await session.rollback()
            raise
```

The session commits automatically on success and rolls back on any exception. Inject into route handlers via `db: AsyncSession = Depends(get_db)`.

### Lifespan Shutdown

Dispose the engine during application shutdown to close all pooled connections:

```python
from contextlib import asynccontextmanager
from fastapi import FastAPI

@asynccontextmanager
async def lifespan(app: FastAPI):
    yield
    await engine.dispose()

app = FastAPI(lifespan=lifespan)
```

---

## 2. Base Model + Mixins

All models inherit from a shared `Base` class. Mixins add reusable columns without inheritance complexity.

```python
# models/base.py
from datetime import datetime
from sqlalchemy import Column, Integer, DateTime, Boolean, func
from sqlalchemy.orm import DeclarativeBase


class Base(DeclarativeBase):
    pass


class TimestampMixin:
    created_at = Column(
        DateTime(timezone=True),
        server_default=func.now(),
        nullable=False,
    )
    updated_at = Column(
        DateTime(timezone=True),
        server_default=func.now(),
        onupdate=func.now(),
        nullable=False,
    )


class SoftDeleteMixin:
    is_deleted = Column(Boolean, default=False, nullable=False)
    deleted_at = Column(DateTime(timezone=True), nullable=True)
```

**TimestampMixin** adds `created_at` and `updated_at` columns. Both use `server_default=func.now()` so the database generates the timestamp. The `onupdate=func.now()` on `updated_at` automatically refreshes on every update.

**SoftDeleteMixin** adds `is_deleted` and `deleted_at` for soft-delete patterns. Queries should filter `is_deleted == False` by default. The `deleted_at` column records when the soft-delete occurred.

### Model Example

```python
# models/user.py
from sqlalchemy import Column, Integer, String, Boolean
from sqlalchemy.orm import relationship
from app.models.base import Base, TimestampMixin


class User(Base, TimestampMixin):
    __tablename__ = "users"

    id = Column(Integer, primary_key=True, index=True)
    email = Column(String(255), unique=True, index=True, nullable=False)
    full_name = Column(String(100), nullable=False)
    hashed_password = Column(String(255), nullable=False)
    is_active = Column(Boolean, default=True, nullable=False)

    items = relationship("Item", back_populates="owner", lazy="selectin")
```

Every entity model should inherit from both `Base` and `TimestampMixin`. Add `SoftDeleteMixin` where soft-delete behavior is needed.

---

## 3. BaseAsyncRepository

A generic repository providing type-safe CRUD operations. All methods are async and operate on an injected `AsyncSession`.

```python
# repositories/base.py
from typing import Generic, List, Optional, Type, TypeVar, Dict, Any
from sqlalchemy import select, func
from sqlalchemy.ext.asyncio import AsyncSession

ModelType = TypeVar("ModelType")


class BaseAsyncRepository(Generic[ModelType]):
    def __init__(self, model: Type[ModelType], session: AsyncSession):
        self.model = model
        self.session = session

    async def get_by_id(self, id: int) -> Optional[ModelType]:
        result = await self.session.execute(
            select(self.model).where(self.model.id == id)
        )
        return result.scalar_one_or_none()

    async def get_all(self, skip: int = 0, limit: int = 100) -> List[ModelType]:
        result = await self.session.execute(
            select(self.model).offset(skip).limit(limit)
        )
        return list(result.scalars().all())

    async def create(self, obj_data: Dict[str, Any]) -> ModelType:
        db_obj = self.model(**obj_data)
        self.session.add(db_obj)
        await self.session.flush()
        await self.session.refresh(db_obj)
        return db_obj

    async def update(self, db_obj: ModelType, update_data: Dict[str, Any]) -> ModelType:
        for field, value in update_data.items():
            if value is not None:
                setattr(db_obj, field, value)
        await self.session.flush()
        await self.session.refresh(db_obj)
        return db_obj

    async def delete(self, db_obj: ModelType) -> None:
        await self.session.delete(db_obj)
        await self.session.flush()

    async def count(self, **filters) -> int:
        query = select(func.count()).select_from(self.model)
        for field, value in filters.items():
            query = query.where(getattr(self.model, field) == value)
        result = await self.session.execute(query)
        return result.scalar_one()

    async def get_paginated(
        self, page: int = 1, page_size: int = 20
    ) -> tuple[List[ModelType], int]:
        total = await self.count()
        result = await self.session.execute(
            select(self.model).offset((page - 1) * page_size).limit(page_size)
        )
        return list(result.scalars().all()), total
```

**Design decisions:**

- `flush()` instead of `commit()` — the session dependency handles commit/rollback. Flushing writes to the database within the transaction so `refresh()` can read back generated values (e.g., auto-increment IDs, server defaults).
- `Generic[ModelType]` with `TypeVar` provides IDE autocompletion and type checking on return values.
- `get_paginated` returns both the page of results and the total count, enabling the frontend to render pagination controls.

---

## 4. Concrete Repository Example

Extend `BaseAsyncRepository` with domain-specific queries. The constructor binds the model type so callers only pass the session.

```python
# repositories/user_repository.py
from typing import Optional
from sqlalchemy import select
from sqlalchemy.ext.asyncio import AsyncSession
from app.models.user import User
from app.repositories.base import BaseAsyncRepository


class UserRepository(BaseAsyncRepository[User]):
    def __init__(self, session: AsyncSession):
        super().__init__(User, session)

    async def get_by_email(self, email: str) -> Optional[User]:
        result = await self.session.execute(
            select(User).where(User.email == email)
        )
        return result.scalar_one_or_none()

    async def get_active_users(self, skip: int = 0, limit: int = 100) -> list[User]:
        result = await self.session.execute(
            select(User)
            .where(User.is_active == True)
            .offset(skip)
            .limit(limit)
        )
        return list(result.scalars().all())
```

All inherited methods (`get_by_id`, `get_all`, `create`, `update`, `delete`, `count`, `get_paginated`) are available with `User` as the bound type. Add only queries that are specific to the domain entity.

---

## 5. Alembic Async Migrations

### alembic.ini Configuration

```ini
[alembic]
script_location = migrations
file_template = %%(year)d%%(month).2d%%(day).2d_%%(hour).2d%%(minute).2d%%(second).2d_%%(slug)s
```

This produces migration files named like `20240115_093045_create_users_table.py`. Timestamp naming ensures migrations sort chronologically and avoids merge conflicts on sequential revision IDs.

### Async env.py

The standard Alembic `env.py` must be adapted for async engines:

```python
# migrations/env.py
import asyncio
from logging.config import fileConfig
from sqlalchemy.ext.asyncio import async_engine_from_config
from sqlalchemy import pool
from alembic import context
from app.config.settings import settings
from app.models.base import Base
from app.models.user import User  # noqa: F401 — needed for autogenerate
from app.models.item import Item  # noqa: F401

config = context.config
config.set_main_option("sqlalchemy.url", settings.DATABASE_URL)
if config.config_file_name is not None:
    fileConfig(config.config_file_name)
target_metadata = Base.metadata


def do_run_migrations(connection):
    context.configure(connection=connection, target_metadata=target_metadata)
    with context.begin_transaction():
        context.run_migrations()


async def run_async_migrations():
    connectable = async_engine_from_config(
        config.get_section(config.config_ini_section, {}),
        prefix="sqlalchemy.",
        poolclass=pool.NullPool,
    )
    async with connectable.connect() as connection:
        await connection.run_sync(do_run_migrations)
    await connectable.dispose()


def run_migrations_online():
    asyncio.run(run_async_migrations())


if context.is_offline_mode():
    url = config.get_main_option("sqlalchemy.url")
    context.configure(
        url=url, target_metadata=target_metadata, literal_binds=True
    )
    with context.begin_transaction():
        context.run_migrations()
else:
    run_migrations_online()
```

**Critical:** Every model must be imported in `env.py` (even if unused) so that `Base.metadata` contains all table definitions for autogenerate. The `# noqa: F401` comment suppresses unused-import linting.

### Common Commands

```bash
# Create a new migration
alembic revision --autogenerate -m "create_users_table"

# Apply all pending migrations
alembic upgrade head

# Rollback one migration
alembic downgrade -1

# Show current revision
alembic current
```

---

## 6. Query Optimization

### Solving N+1 Queries

The N+1 problem occurs when accessing a relationship triggers a separate query for each parent row. Two strategies prevent this:

**selectinload** — Loads related objects in a second `SELECT ... WHERE id IN (...)` query. Best for one-to-many relationships.

```python
from sqlalchemy.orm import selectinload

async def get_users_with_items(session: AsyncSession) -> list[User]:
    result = await session.execute(
        select(User).options(selectinload(User.items))
    )
    return list(result.scalars().all())
```

**joinedload** — Loads related objects via a `JOIN` in a single query. Best for many-to-one or one-to-one relationships where the related table is small.

```python
from sqlalchemy.orm import joinedload

async def get_items_with_owner(session: AsyncSession) -> list[Item]:
    result = await session.execute(
        select(Item).options(joinedload(Item.owner))
    )
    return list(result.unique().scalars().all())
```

**When to use which:**

| Strategy | Best For | Trade-off |
|----------|----------|-----------|
| `selectinload` | One-to-many, large child collections | Two queries, but avoids row multiplication |
| `joinedload` | Many-to-one, one-to-one | Single query, but multiplies rows for large collections |

### Bulk Operations

For inserting or updating many records, use bulk methods to minimize round-trips:

```python
async def bulk_create_users(session: AsyncSession, users_data: list[dict]) -> None:
    db_objects = [User(**data) for data in users_data]
    session.add_all(db_objects)
    await session.flush()
```

### General Rules

1. Set `lazy="selectin"` on relationship definitions for default eager loading.
2. Use explicit `.options(selectinload(...))` when you need control per-query.
3. Index all columns used in `WHERE`, `ORDER BY`, and `JOIN` clauses: `Column(String, index=True)`.
4. Use `func.count()` for counting rows — never `len(results)` which loads all rows into memory.
5. Paginate all list endpoints. Never return unbounded result sets.

---

## 7. Model Relationships

### One-to-Many

A user owns many items. Define `relationship` on both sides with `back_populates`.

```python
# models/user.py
class User(Base, TimestampMixin):
    __tablename__ = "users"

    id = Column(Integer, primary_key=True, index=True)
    email = Column(String(255), unique=True, index=True, nullable=False)

    items = relationship("Item", back_populates="owner", lazy="selectin")


# models/item.py
from sqlalchemy import ForeignKey

class Item(Base, TimestampMixin):
    __tablename__ = "items"

    id = Column(Integer, primary_key=True, index=True)
    title = Column(String(200), nullable=False)
    owner_id = Column(Integer, ForeignKey("users.id"), nullable=False)

    owner = relationship("User", back_populates="items", lazy="selectin")
```

The `ForeignKey("users.id")` on `owner_id` creates the database-level constraint. The `relationship` declarations provide the ORM-level navigation.

### Many-to-Many with Association Table

For many-to-many relationships, define an explicit association table:

```python
# models/associations.py
from sqlalchemy import Table, Column, Integer, ForeignKey
from app.models.base import Base

user_roles = Table(
    "user_roles",
    Base.metadata,
    Column("user_id", Integer, ForeignKey("users.id"), primary_key=True),
    Column("role_id", Integer, ForeignKey("roles.id"), primary_key=True),
)


# models/user.py
class User(Base, TimestampMixin):
    __tablename__ = "users"

    id = Column(Integer, primary_key=True, index=True)
    email = Column(String(255), unique=True, index=True, nullable=False)

    roles = relationship("Role", secondary=user_roles, back_populates="users", lazy="selectin")


# models/role.py
class Role(Base, TimestampMixin):
    __tablename__ = "roles"

    id = Column(Integer, primary_key=True, index=True)
    name = Column(String(50), unique=True, nullable=False)

    users = relationship("User", secondary=user_roles, back_populates="roles", lazy="selectin")
```

The `secondary=user_roles` parameter tells SQLAlchemy to use the association table for the join. Both sides define `back_populates` pointing to the other.

### Querying Many-to-Many

```python
async def get_user_with_roles(session: AsyncSession, user_id: int) -> Optional[User]:
    result = await session.execute(
        select(User)
        .options(selectinload(User.roles))
        .where(User.id == user_id)
    )
    return result.scalar_one_or_none()

async def assign_role_to_user(session: AsyncSession, user: User, role: Role) -> None:
    user.roles.append(role)
    await session.flush()
```

With `lazy="selectin"` set on the relationship, accessing `user.roles` will automatically load the related roles without an explicit `.options()` call. Use explicit `selectinload` when you need to override the default or load nested relationships.
