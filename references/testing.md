# Testing Reference

## Test Structure

Tests are organized by type. Unit tests validate business logic in isolation using fake dependencies. Integration tests exercise the full HTTP stack through the FastAPI application.

### Simple Project Layout

```
tests/
├── conftest.py
├── unit/
│   └── test_user_service.py
├── integration/
│   └── test_user_endpoints.py
└── utils/
    ├── fakes.py              # FakeRepository classes
    └── assertions.py         # Response format helpers
```

### Modular Project Layout

For applications with multiple feature modules, mirror the module structure inside the tests directory:

```
tests/
├── conftest.py
├── modules/
│   ├── auth/
│   │   └── test_auth_endpoints.py
│   └── users/
│       ├── test_user_service.py
│       └── test_user_endpoints.py
└── utils/
    ├── fakes.py
    └── assertions.py
```

The `conftest.py` at the root provides shared fixtures (database session, HTTP client, test user). Module-specific `conftest.py` files can be added for fixtures that only apply to one module.

The `utils/` directory is not a test package -- it holds reusable fakes and assertion helpers that tests import.

---

## Core conftest.py

The root `conftest.py` defines all shared async fixtures. Every fixture is async-native and uses dependency injection.

```python
# tests/conftest.py
import pytest
from typing import AsyncGenerator
from httpx import ASGITransport, AsyncClient
from sqlalchemy.ext.asyncio import (
    AsyncSession,
    async_sessionmaker,
    create_async_engine,
)

from app.main import app
from app.config.database import get_db
from app.models.base import Base
from app.core.security import create_access_token, hash_password

TEST_DATABASE_URL = "sqlite+aiosqlite:///./test.db"

test_engine = create_async_engine(TEST_DATABASE_URL, echo=True)
TestSession = async_sessionmaker(
    bind=test_engine, class_=AsyncSession, expire_on_commit=False
)


@pytest.fixture(scope="session")
def event_loop():
    import asyncio
    loop = asyncio.get_event_loop_policy().new_event_loop()
    yield loop
    loop.close()


@pytest.fixture(autouse=True)
async def setup_db():
    async with test_engine.begin() as conn:
        await conn.run_sync(Base.metadata.create_all)
    yield
    async with test_engine.begin() as conn:
        await conn.run_sync(Base.metadata.drop_all)


@pytest.fixture
async def db_session() -> AsyncGenerator[AsyncSession, None]:
    async with TestSession() as session:
        yield session


@pytest.fixture
async def client(db_session: AsyncSession) -> AsyncGenerator[AsyncClient, None]:
    async def override_get_db():
        yield db_session

    app.dependency_overrides[get_db] = override_get_db
    async with AsyncClient(
        transport=ASGITransport(app=app), base_url="http://test"
    ) as ac:
        yield ac
    app.dependency_overrides.clear()


@pytest.fixture
def user_data() -> dict:
    return {
        "email": "test@example.com",
        "full_name": "Test User",
        "password": "Test1234",
    }


@pytest.fixture
async def created_user(db_session: AsyncSession, user_data: dict):
    from app.models.User import User
    user = User(
        email=user_data["email"],
        full_name=user_data["full_name"],
        hashed_password=hash_password(user_data["password"]),
        is_active=True,
    )
    db_session.add(user)
    await db_session.commit()
    await db_session.refresh(user)
    return user


@pytest.fixture
async def auth_client(client: AsyncClient, created_user) -> AsyncClient:
    token = create_access_token(created_user.id)
    client.headers["Authorization"] = f"Bearer {token}"
    return client
```

### Fixture Dependency Chain

The fixtures form a clear dependency graph:

- `setup_db` runs automatically before every test. It creates all tables, yields control to the test, then drops all tables. This guarantees a clean database for each test.
- `db_session` opens a new async session scoped to a single test function.
- `client` depends on `db_session`. It overrides the `get_db` dependency so the FastAPI app uses the test session instead of the real database. After the test, `dependency_overrides` is cleared.
- `created_user` depends on `db_session` and `user_data`. It inserts a user directly via the ORM, bypassing the API layer. This is useful for integration tests that need an existing user as a precondition.
- `auth_client` depends on `client` and `created_user`. It attaches a valid JWT Bearer token to the client headers.

Dependencies: `poetry add --group dev pytest pytest-asyncio httpx aiosqlite`

---

## Test Database Setup

Tests use SQLite with `aiosqlite` instead of PostgreSQL. This avoids the need for a running database server in CI and keeps test execution fast.

```python
TEST_DATABASE_URL = "sqlite+aiosqlite:///./test.db"
test_engine = create_async_engine(TEST_DATABASE_URL, echo=True)
```

The `setup_db` fixture (shown above) creates all tables before each test and drops them after. This isolation strategy means tests never see leftover data from previous runs.

### Dependency Override

The core mechanism that connects the test database to the FastAPI app is `dependency_overrides`:

```python
app.dependency_overrides[get_db] = override_get_db
```

This tells FastAPI to use the test session whenever a route or dependency calls `get_db`. The override is scoped to the `client` fixture and cleared after each test. No application code needs to change -- the same `Depends(get_db)` works in production and tests.

### SQLite Limitations

SQLite does not support all PostgreSQL features (array columns, `JSONB`, concurrent writes). For tests that rely on PostgreSQL-specific behavior, use a PostgreSQL test container via `testcontainers-python` or a dedicated test database. For the vast majority of CRUD tests, SQLite is sufficient and significantly faster.

---

## FakeRepository Pattern

Because services depend on Protocol classes rather than concrete repository implementations, you can substitute a simple in-memory fake for unit tests. No mocking libraries needed.

```python
# tests/utils/fakes.py
from app.models.User import User


class FakeUserRepository:
    """In-memory repo satisfying UserRepositoryProtocol."""

    def __init__(self):
        self.users: dict[int, User] = {}
        self._next_id = 1

    async def get_by_id(self, id: int) -> User | None:
        return self.users.get(id)

    async def get_by_email(self, email: str) -> User | None:
        return next(
            (u for u in self.users.values() if u.email == email), None
        )

    async def create(self, obj_data: dict) -> User:
        user = User(id=self._next_id, **obj_data)
        self.users[user.id] = user
        self._next_id += 1
        return user

    async def update(self, db_obj: User, update_data: dict) -> User:
        for key, val in update_data.items():
            if val is not None:
                setattr(db_obj, key, val)
        return db_obj

    async def delete(self, db_obj: User) -> None:
        self.users.pop(db_obj.id, None)

    async def count(self, **filters) -> int:
        return len(self.users)

    async def get_active_users(self) -> list[User]:
        return [u for u in self.users.values() if u.is_active]
```

### Why Fakes Over Mocks

- **Type safety** -- The fake implements the same Protocol as the real repository. If the Protocol changes (e.g., a method is renamed), the fake breaks at test time, catching the inconsistency early. `AsyncMock` does not enforce method signatures.
- **Readable tests** -- Setting up state means calling `await repo.create(...)`, which reads like normal application code. No `mock.return_value = ...` chains.
- **Behavioral correctness** -- The fake maintains internal state (the `users` dict). Calling `create` followed by `get_by_email` returns the created user, just like the real repository. Mocks do not provide this unless you wire up side effects manually.
- **Reusability** -- One `FakeUserRepository` class is shared across every unit test for the user service. With mocks, each test typically configures its own mock instance.

To create a fake for a new entity, copy the structure of `FakeUserRepository`, change the model type, and implement the methods defined in the corresponding Protocol.

---

## Unit Tests

Unit tests validate service-layer logic using `FakeRepository` instances. They follow the Arrange-Act-Assert pattern.

```python
# tests/unit/test_user_service.py
import pytest
from app.services.UserService import UserService
from app.core.exceptions import NotFoundError, ConflictError
from tests.utils.fakes import FakeUserRepository


class TestUserService:
    @pytest.fixture
    def repo(self):
        return FakeUserRepository()

    @pytest.fixture
    def service(self, repo):
        return UserService(repo)

    async def test_get_by_id_found(self, service, repo):
        # Arrange
        await repo.create({
            "email": "a@b.com",
            "full_name": "Alice",
            "hashed_password": "x",
        })

        # Act
        user = await service.get_by_id(1)

        # Assert
        assert user.email == "a@b.com"

    async def test_get_by_id_not_found(self, service):
        with pytest.raises(NotFoundError):
            await service.get_by_id(999)

    async def test_create_user_success(self, service):
        from unittest.mock import MagicMock
        data = MagicMock()
        data.email = "new@b.com"
        data.model_dump.return_value = {
            "email": "new@b.com",
            "full_name": "New",
            "hashed_password": "x",
        }

        user = await service.create_user(data)
        assert user.email == "new@b.com"

    async def test_create_duplicate_email(self, service, repo):
        await repo.create({
            "email": "dup@b.com",
            "full_name": "Dup",
            "hashed_password": "x",
        })
        from unittest.mock import MagicMock
        data = MagicMock()
        data.email = "dup@b.com"

        with pytest.raises(ConflictError):
            await service.create_user(data)
```

### Key Characteristics

- **No database** -- Tests run entirely in memory via `FakeUserRepository`. No fixtures for `db_session` or `client` needed.
- **Fast execution** -- Without I/O, unit tests complete in milliseconds.
- **Focused assertions** -- Each test validates one behavior. `test_get_by_id_not_found` only checks that the correct exception is raised. It does not test the response format or HTTP status code -- those are integration test concerns.
- **`asyncio_mode = "auto"`** -- With this pytest configuration (see below), there is no need to decorate every test with `@pytest.mark.asyncio`. Async test functions are detected automatically.

---

## Integration Tests

Integration tests send real HTTP requests through the full FastAPI stack using `httpx.AsyncClient`. They validate the 5-field response format: `success`, `code`, `message`, `data`, and `errors`.

```python
# tests/integration/test_user_endpoints.py
import pytest
from httpx import AsyncClient


class TestUserEndpoints:
    async def test_create_user(self, client: AsyncClient, user_data):
        response = await client.post("/api/v1/users/", json=user_data)
        assert response.status_code == 201
        body = response.json()

        # Verify standard response format
        assert body["success"] is True
        assert body["code"] == 201
        assert body["data"]["email"] == user_data["email"]
        assert body["errors"] is None
        assert "password" not in body["data"]

    async def test_create_user_duplicate(
        self, client: AsyncClient, created_user, user_data
    ):
        response = await client.post("/api/v1/users/", json=user_data)
        assert response.status_code == 409
        body = response.json()
        assert body["success"] is False
        assert body["errors"] is None  # not a validation error

    async def test_validation_error_format(self, client: AsyncClient):
        response = await client.post("/api/v1/users/", json={"email": "bad"})
        assert response.status_code == 422
        body = response.json()

        assert body["success"] is False
        assert body["code"] == 422
        assert body["message"] == "Validation failed"
        assert body["data"] is None
        assert isinstance(body["errors"], list)
        # Each error has field + message
        for err in body["errors"]:
            assert "field" in err
            assert "message" in err
            assert "body." not in err["field"]  # no ugly prefix

    async def test_get_me_authenticated(self, auth_client: AsyncClient):
        response = await auth_client.get("/api/v1/users/me")
        assert response.status_code == 200
        assert response.json()["success"] is True

    async def test_get_me_unauthenticated(self, client: AsyncClient):
        response = await client.get("/api/v1/users/me")
        assert response.status_code == 401
        assert response.json()["success"] is False
```

### What Integration Tests Validate

1. **HTTP status codes** -- The correct status code is returned for success, conflict, validation error, and authentication failure.
2. **Response shape** -- Every response contains all 5 fields (`success`, `code`, `message`, `data`, `errors`) in the expected types.
3. **Data correctness** -- The `data` field contains the expected payload (e.g., the created user's email) and excludes sensitive fields (e.g., `password` is never in the response).
4. **Error format** -- Validation errors produce a list of `{field, message}` objects. The `field` values are clean (no `body.` prefix from Pydantic's internal path).
5. **Authentication** -- Protected endpoints return 401 without a token and 200 with a valid token.

---

## Response Format Assertion Helpers

To reduce repetition in integration tests, use helper functions that validate the standard response envelope.

```python
# tests/utils/assertions.py

def assert_success_response(response, expected_code: int = 200):
    """Assert a successful response with the standard 5-field format."""
    body = response.json()
    assert response.status_code == expected_code
    assert body["success"] is True
    assert body["code"] == expected_code
    assert body["errors"] is None


def assert_error_response(response, expected_code: int):
    """Assert an error response with the standard 5-field format."""
    body = response.json()
    assert response.status_code == expected_code
    assert body["success"] is False
    assert body["code"] == expected_code


def assert_validation_error(response):
    """Assert a 422 validation error with field-level error details."""
    body = response.json()
    assert response.status_code == 422
    assert body["success"] is False
    assert isinstance(body["errors"], list)
    assert len(body["errors"]) > 0
    for err in body["errors"]:
        assert "field" in err and "message" in err
```

### Usage in Tests

```python
async def test_create_user(self, client, user_data):
    response = await client.post("/api/v1/users/", json=user_data)
    assert_success_response(response, 201)
    assert response.json()["data"]["email"] == user_data["email"]

async def test_not_found(self, client):
    response = await client.get("/api/v1/users/999")
    assert_error_response(response, 404)

async def test_invalid_input(self, client):
    response = await client.post("/api/v1/users/", json={"email": "bad"})
    assert_validation_error(response)
```

These helpers enforce the response contract consistently. If the response format changes, you update the helpers in one place.

---

## pytest Configuration

All pytest and coverage settings go in `pyproject.toml`:

```toml
[tool.pytest.ini_options]
testpaths = ["tests"]
asyncio_mode = "auto"
addopts = "-v --tb=short -x"
markers = [
    "slow: slow tests",
    "integration: integration tests",
    "unit: unit tests",
]

[tool.coverage.run]
source = ["app"]
omit = ["app/config/*", "tests/*", "migrations/*"]

[tool.coverage.report]
fail_under = 80
show_missing = true
```

### Key Settings

- **`asyncio_mode = "auto"`** -- Async test functions are detected and run automatically. No need to add `@pytest.mark.asyncio` to every test. This eliminates a common source of test failures where the decorator is accidentally omitted.
- **`addopts = "-v --tb=short -x"`** -- Verbose output, short tracebacks, stop on first failure. The `-x` flag is useful during development; remove it in CI if you want to see all failures.
- **`fail_under = 80`** -- The coverage report fails the build if total coverage drops below 80%. This is a minimum threshold -- aim higher for critical paths like authentication and payment processing.
- **`show_missing = true`** -- The coverage report includes line numbers for uncovered code, making it easy to identify what needs tests.
- **`omit`** -- Configuration files, test files themselves, and Alembic migrations are excluded from coverage calculations. These are infrastructure code that does not benefit from unit test coverage.

### Running Tests

```bash
# Run all tests
pytest

# Run only unit tests
pytest tests/unit/

# Run only integration tests
pytest tests/integration/

# Run with coverage report
pytest --cov=app --cov-report=html

# Run a specific test class
pytest tests/unit/test_user_service.py::TestUserService

# Run a specific test method
pytest tests/unit/test_user_service.py::TestUserService::test_get_by_id_found
```

The `--cov-report=html` flag generates an HTML coverage report in `htmlcov/`. Open `htmlcov/index.html` in a browser to see line-by-line coverage for every source file.
