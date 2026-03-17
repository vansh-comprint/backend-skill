# Security Reference

JWT authentication, password hashing, RBAC, CORS, security headers, rate limiting, and audit checklist for FastAPI. Python 3.12+, all auth code async, Annotated dependency injection.

---

## 1. JWT Token Creation & Verification

Two-token strategy: a short-lived access token for API requests and a long-lived refresh token for obtaining new access tokens without re-entering credentials.

| Token | Lifetime | Purpose |
|-------|----------|---------|
| Access token | 15-30 minutes | Sent in `Authorization: Bearer <token>` on every request |
| Refresh token | 7 days | Exchanged for a new access token when the current one expires |

Both tokens are signed with HS256 using `python-jose` (or `PyJWT` as an alternative). The `exp` claim is always set and always verified on decode.

```python
# core/security.py
from datetime import datetime, timedelta, timezone
from typing import Any

from jose import JWTError, jwt
from passlib.context import CryptContext
from fastapi import Depends, Security
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials
from sqlalchemy.ext.asyncio import AsyncSession

from app.config.settings import settings
from app.config.database import get_db
from app.core.exceptions import UnauthorizedError, ForbiddenError
from app.models.User import User
from app.repositories.UserRepository import UserRepository

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")
security_scheme = HTTPBearer()


def create_access_token(subject: int, extra: dict[str, Any] | None = None) -> str:
    """Create a short-lived access token with an exp claim."""
    expire = datetime.now(timezone.utc) + timedelta(minutes=settings.ACCESS_TOKEN_EXPIRE_MINUTES)
    claims = {"sub": str(subject), "exp": expire, "type": "access"}
    if extra:
        claims.update(extra)
    return jwt.encode(claims, settings.SECRET_KEY, algorithm=settings.ALGORITHM)


def create_refresh_token(subject: int) -> str:
    """Create a long-lived refresh token for silent re-auth."""
    expire = datetime.now(timezone.utc) + timedelta(days=settings.REFRESH_TOKEN_EXPIRE_DAYS)
    claims = {"sub": str(subject), "exp": expire, "type": "refresh"}
    return jwt.encode(claims, settings.SECRET_KEY, algorithm=settings.ALGORITHM)


def verify_token(token: str, token_type: str = "access") -> dict[str, Any]:
    """Decode and validate a JWT. Raises UnauthorizedError on failure."""
    try:
        payload = jwt.decode(token, settings.SECRET_KEY, algorithms=[settings.ALGORITHM])
        if payload.get("type") != token_type:
            raise JWTError("Invalid token type")
        return payload
    except JWTError:
        raise UnauthorizedError("Invalid or expired token")
```

**Settings required** (in `config/settings.py` or `.env`):

```python
SECRET_KEY: str          # At least 32 characters, randomly generated
ALGORITHM: str = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES: int = 30
REFRESH_TOKEN_EXPIRE_DAYS: int = 7
```

**Key rules:**

- Never store tokens in the database unless you need server-side revocation. If you do, store a hash of the refresh token, not the raw value.
- The `sub` claim always holds the user ID as a string. Parse it back to `int` when looking up the user.
- The `type` claim prevents an attacker from using a refresh token as an access token.

---

## 2. Password Hashing

All password hashing uses bcrypt via `passlib`. Never use MD5, SHA-1, or SHA-256 for passwords — they are too fast and vulnerable to brute force.

```python
# core/security.py (included in the module above)
pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")


def hash_password(password: str) -> str:
    """Hash a plaintext password with bcrypt. Returns the full bcrypt hash string."""
    return pwd_context.hash(password)


def verify_password(plain: str, hashed: str) -> bool:
    """Compare a plaintext password against a bcrypt hash. Returns True if they match."""
    return pwd_context.verify(plain, hashed)
```

**Usage rules:**

- Call `hash_password()` once during registration. Store the result in `user.hashed_password`.
- Call `verify_password()` during login. Never compare password strings directly.
- The `deprecated="auto"` setting tells passlib to automatically re-hash passwords if the bcrypt version or cost factor changes, keeping stored hashes up to date.
- Never log, return, or serialize the hashed password. Exclude it from all Pydantic response schemas.

---

## 3. RBAC Models

Role-Based Access Control links users to roles, and roles to permissions, through many-to-many association tables. This gives you fine-grained control without hardcoding permission checks.

```python
# models/rbac.py
from sqlalchemy import Column, Integer, String, ForeignKey, Table
from sqlalchemy.orm import relationship
from app.config.database import Base

user_roles = Table(
    "user_roles",
    Base.metadata,
    Column("user_id", Integer, ForeignKey("users.id", ondelete="CASCADE"), primary_key=True),
    Column("role_id", Integer, ForeignKey("roles.id", ondelete="CASCADE"), primary_key=True),
)

role_permissions = Table(
    "role_permissions",
    Base.metadata,
    Column("role_id", Integer, ForeignKey("roles.id", ondelete="CASCADE"), primary_key=True),
    Column("permission_id", Integer, ForeignKey("permissions.id", ondelete="CASCADE"), primary_key=True),
)


class Role(Base):
    __tablename__ = "roles"

    id = Column(Integer, primary_key=True)
    name = Column(String(50), unique=True, nullable=False)
    permissions = relationship("Permission", secondary=role_permissions, lazy="selectin")


class Permission(Base):
    __tablename__ = "permissions"

    id = Column(Integer, primary_key=True)
    name = Column(String(100), unique=True, nullable=False)  # e.g. "users:delete", "posts:create"
```

Add the relationship to the `User` model:

```python
# models/User.py (add to existing model)
class User(Base):
    __tablename__ = "users"

    id = Column(Integer, primary_key=True)
    email = Column(String(255), unique=True, nullable=False)
    hashed_password = Column(String(255), nullable=False)
    is_active = Column(Boolean, default=True)
    roles = relationship("Role", secondary=user_roles, lazy="selectin")

    def has_role(self, role_name: str) -> bool:
        """Check if user has a specific role by name."""
        return any(r.name == role_name for r in self.roles)

    def has_permission(self, permission_name: str) -> bool:
        """Check if any of the user's roles grant a specific permission."""
        return any(
            p.name == permission_name
            for role in self.roles
            for p in role.permissions
        )
```

**Permission naming convention:** Use the format `resource:action` (e.g. `users:delete`, `posts:create`, `reports:export`). This makes permissions grep-able and predictable.

**Checking permissions in endpoints:**

```python
async def require_permission(permission: str):
    """Factory that returns a dependency requiring a specific permission."""
    async def _check(user: CurrentUser):
        if not user.has_permission(permission):
            raise ForbiddenError(f"Missing permission: {permission}")
        return user
    return _check

# Usage on a route:
@router.delete(
    "/users/{user_id}",
    dependencies=[Depends(require_permission("users:delete"))],
)
async def delete_user(user_id: int, service: UserServiceDep):
    await service.delete_user(user_id)
    return StandardResponse(message="User deleted")
```

---

## 4. Auth Dependencies

FastAPI's `Annotated` type aliases keep endpoint signatures clean. Define them once in `core/security.py` and import everywhere.

```python
# core/security.py (continued)
from typing import Annotated


async def get_current_user_from_token(
    credentials: HTTPAuthorizationCredentials = Security(security_scheme),
    db: AsyncSession = Depends(get_db),
) -> User:
    """Extract the Bearer token, verify it, and return the active user."""
    payload = verify_token(credentials.credentials, token_type="access")
    user_id = int(payload["sub"])
    user = await UserRepository(db).get_by_id(user_id)

    if not user:
        raise UnauthorizedError("User not found")
    if not user.is_active:
        raise UnauthorizedError("Account deactivated")
    return user


async def require_admin(
    user: User = Depends(get_current_user_from_token),
) -> User:
    """Reject non-admin users with a 403."""
    if not user.is_admin:
        raise ForbiddenError("Admin access required")
    return user


# Type aliases — use these in endpoint signatures
CurrentUser = Annotated[User, Depends(get_current_user_from_token)]
AdminUser = Annotated[User, Depends(require_admin)]
```

**Usage in endpoints:**

```python
@router.get("/me")
async def get_me(user: CurrentUser):
    """Any authenticated user can access this."""
    return StandardResponse(data=user)


@router.delete("/users/{user_id}")
async def delete_user(user_id: int, admin: AdminUser, service: UserServiceDep):
    """Only admins can access this."""
    await service.delete_user(user_id)
    return StandardResponse(message="User deleted")
```

The dependency chain is: `HTTPBearer` extracts the token from the `Authorization` header, `get_current_user_from_token` verifies it and loads the user, and `require_admin` adds the admin check on top. FastAPI resolves the entire chain automatically.

---

## 5. Auth Endpoints

Three endpoints: register, login, and refresh. All return the same `TokenResponse` shape so the frontend handles them uniformly.

```python
# api/v1/endpoints/auth.py
from fastapi import APIRouter, Depends
from app.api.dependencies import DbSession
from app.core.security import (
    verify_password, hash_password,
    create_access_token, create_refresh_token, verify_token,
)
from app.core.exceptions import UnauthorizedError, ConflictError
from app.schemas.AuthSchema import LoginRequest, RegisterRequest, TokenResponse, RefreshRequest
from app.schemas.common import StandardResponse
from app.repositories.UserRepository import UserRepository

router = APIRouter()


@router.post("/register", response_model=StandardResponse[TokenResponse], status_code=201)
async def register(data: RegisterRequest, db: DbSession):
    repo = UserRepository(db)
    if await repo.get_by_email(data.email):
        raise ConflictError("User", data.email)

    user = await repo.create({
        "email": data.email,
        "full_name": data.full_name,
        "hashed_password": hash_password(data.password),
    })
    tokens = TokenResponse(
        access_token=create_access_token(user.id),
        refresh_token=create_refresh_token(user.id),
    )
    return StandardResponse(code=201, data=tokens, message="Registration successful")


@router.post("/login", response_model=StandardResponse[TokenResponse])
async def login(data: LoginRequest, db: DbSession):
    repo = UserRepository(db)
    user = await repo.get_by_email(data.email)
    if not user or not verify_password(data.password, user.hashed_password):
        raise UnauthorizedError("Invalid email or password")

    tokens = TokenResponse(
        access_token=create_access_token(user.id),
        refresh_token=create_refresh_token(user.id),
    )
    return StandardResponse(data=tokens, message="Login successful")


@router.post("/refresh", response_model=StandardResponse[TokenResponse])
async def refresh(data: RefreshRequest, db: DbSession):
    """Exchange a valid refresh token for a new access + refresh token pair."""
    payload = verify_token(data.refresh_token, token_type="refresh")
    user_id = int(payload["sub"])

    repo = UserRepository(db)
    user = await repo.get_by_id(user_id)
    if not user or not user.is_active:
        raise UnauthorizedError("User not found or deactivated")

    tokens = TokenResponse(
        access_token=create_access_token(user.id),
        refresh_token=create_refresh_token(user.id),
    )
    return StandardResponse(data=tokens, message="Token refreshed")
```

**Auth schemas:**

```python
# schemas/AuthSchema.py
from pydantic import BaseModel, EmailStr, Field, field_validator


class LoginRequest(BaseModel):
    email: EmailStr
    password: str


class RegisterRequest(BaseModel):
    email: EmailStr
    full_name: str = Field(..., min_length=1, max_length=100)
    password: str = Field(..., min_length=8, max_length=128)

    @field_validator("password")
    @classmethod
    def strong_password(cls, v: str) -> str:
        if not any(c.isupper() for c in v):
            raise ValueError("Must contain at least one uppercase letter")
        if not any(c.isdigit() for c in v):
            raise ValueError("Must contain at least one digit")
        return v


class RefreshRequest(BaseModel):
    refresh_token: str


class TokenResponse(BaseModel):
    access_token: str
    refresh_token: str
    token_type: str = "bearer"
```

**Security notes on auth endpoints:**

- The login endpoint returns the same error message for "user not found" and "wrong password". This prevents email enumeration.
- The refresh endpoint issues both a new access token and a new refresh token (token rotation). This limits the window if a refresh token is compromised.
- Always apply rate limiting to `/register`, `/login`, and `/refresh` (see section 8).

---

## 6. CORS Configuration

Cross-Origin Resource Sharing controls which frontends can call your API. Misconfigured CORS is one of the most common security issues in new projects.

```python
# main.py
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=settings.ALLOWED_ORIGINS,   # never ["*"] in production
    allow_credentials=True,
    allow_methods=settings.ALLOWED_METHODS,
    allow_headers=settings.ALLOWED_HEADERS,
)
```

**Settings example:**

```python
# config/settings.py
ALLOWED_ORIGINS: list[str] = [
    "https://app.example.com",
    "https://admin.example.com",
]
ALLOWED_METHODS: list[str] = ["GET", "POST", "PUT", "PATCH", "DELETE"]
ALLOWED_HEADERS: list[str] = ["Authorization", "Content-Type"]
```

**Rules:**

- Never use `allow_origins=["*"]` in production. It disables credential support and opens the API to any origin.
- For local development, add `http://localhost:3000` (or your frontend dev port) to the origins list via environment variables. Do not hardcode development origins in production config.
- `allow_credentials=True` is required when the frontend sends cookies or `Authorization` headers. It is incompatible with wildcard origins — FastAPI will raise an error if you combine them.
- Keep `allow_methods` explicit. Listing only the HTTP methods your API actually uses reduces attack surface.

---

## 7. Security Headers Middleware

Security headers instruct browsers to enable built-in protections. Add them as middleware so every response includes them automatically.

```python
# core/middleware.py
from starlette.middleware.base import BaseHTTPMiddleware
from starlette.requests import Request
from starlette.responses import Response


class SecurityHeadersMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request: Request, call_next) -> Response:
        response = await call_next(request)
        response.headers["X-Content-Type-Options"] = "nosniff"
        response.headers["X-Frame-Options"] = "DENY"
        response.headers["X-XSS-Protection"] = "1; mode=block"
        response.headers["Strict-Transport-Security"] = "max-age=31536000; includeSubDomains"
        response.headers["Referrer-Policy"] = "strict-origin-when-cross-origin"
        response.headers["Content-Security-Policy"] = "default-src 'self'"
        return response
```

**Register in `main.py`:**

```python
from app.core.middleware import SecurityHeadersMiddleware

app.add_middleware(SecurityHeadersMiddleware)
```

| Header | What it does |
|--------|-------------|
| `X-Content-Type-Options: nosniff` | Prevents browsers from MIME-sniffing a response away from the declared Content-Type |
| `X-Frame-Options: DENY` | Blocks the page from being embedded in iframes (clickjacking protection) |
| `Strict-Transport-Security` | Forces HTTPS for the specified duration. Only set this when your domain is fully on HTTPS |
| `Referrer-Policy` | Controls how much referrer information is sent with requests |
| `Content-Security-Policy` | Restricts the sources from which the browser can load resources. Tune the value based on what your frontend needs |

**Note:** If your FastAPI application serves only JSON API responses (no HTML), `X-XSS-Protection` and `Content-Security-Policy` are less critical but still worth setting as defense-in-depth.

---

## 8. Rate Limiting

Redis-based rate limiting prevents brute-force attacks on auth endpoints and protects expensive operations. The implementation uses a sliding window counter per IP and path.

```python
# core/rate_limit.py
from fastapi import Depends, Request
from redis.asyncio import Redis
from app.config.settings import settings
from app.core.exceptions import RateLimitError

redis_client = Redis.from_url(settings.REDIS_URL)


def rate_limit(limit: int = 100, window: int = 60):
    """
    Return a FastAPI dependency that enforces rate limiting.

    Args:
        limit: Maximum number of requests allowed within the window.
        window: Time window in seconds.
    """
    async def _limiter(request: Request):
        client_ip = request.client.host if request.client else "unknown"
        key = f"rl:{client_ip}:{request.url.path}"
        current = await redis_client.incr(key)
        if current == 1:
            await redis_client.expire(key, window)
        if current > limit:
            raise RateLimitError()
    return _limiter
```

**Per-endpoint configuration:**

```python
# Auth endpoints — strict limits
@router.post("/login", dependencies=[Depends(rate_limit(limit=10, window=60))])
async def login(data: LoginRequest, db: DbSession):
    ...

@router.post("/register", dependencies=[Depends(rate_limit(limit=5, window=60))])
async def register(data: RegisterRequest, db: DbSession):
    ...

@router.post("/forgot-password", dependencies=[Depends(rate_limit(limit=3, window=300))])
async def forgot_password(data: ForgotPasswordRequest, db: DbSession):
    ...

# General API endpoints — relaxed limits
@router.get("/users", dependencies=[Depends(rate_limit(limit=100, window=60))])
async def list_users(service: UserServiceDep):
    ...
```

**The `RateLimitError` exception** (add to `core/exceptions.py`):

```python
class RateLimitError(BaseAPIException):
    def __init__(self):
        super().__init__(
            status_code=429,
            message="Too many requests. Please try again later.",
            headers={"Retry-After": "60"},
        )
```

**Recommended limits:**

| Endpoint | Limit | Window |
|----------|-------|--------|
| `/login` | 10 requests | 60 seconds |
| `/register` | 5 requests | 60 seconds |
| `/forgot-password` | 3 requests | 300 seconds |
| General API | 100 requests | 60 seconds |

**Production considerations:**

- Behind a reverse proxy (nginx, AWS ALB), `request.client.host` may be the proxy IP. Configure your proxy to set `X-Forwarded-For` and use FastAPI's `TrustedHostMiddleware` or read the header directly.
- For distributed deployments, Redis is the correct backing store since it is shared across all application instances. An in-memory counter would only limit per-process.

---

## 9. Security Audit Checklist

Concrete, grep-able checks to run before every deployment. Each item can be verified with a code search or a quick test.

- [ ] **JWT has `exp` claim and is verified** — `verify_token()` calls `jwt.decode()` which rejects expired tokens by default. Confirm `exp` is set in both `create_access_token()` and `create_refresh_token()`. Grep: `"exp"` in `core/security.py`.

- [ ] **Passwords hashed with bcrypt (not MD5/SHA)** — `CryptContext(schemes=["bcrypt"])` is the only hashing configuration. Grep for `md5`, `sha1`, or `sha256` in Python files — there should be zero matches in auth code.

- [ ] **No hardcoded secrets** — `SECRET_KEY`, database passwords, and API keys come from environment variables or a secrets manager. Grep for `SECRET`, `PASSWORD`, `API_KEY` in Python files and verify every match references `settings` or `os.environ`, never a string literal.

- [ ] **SQL uses parameterized statements (no f-strings)** — All database queries go through SQLAlchemy's query builder or use `text()` with bound parameters. Grep for f-string SQL patterns like `f"SELECT` or `f"INSERT` — there should be zero matches.

- [ ] **CORS origins explicit (no wildcard in production)** — `allow_origins` is set to a list of specific domains, not `["*"]`. Check `settings.ALLOWED_ORIGINS` and the `CORSMiddleware` call in `main.py`.

- [ ] **Sensitive data not logged** — Passwords, tokens, and PII are never passed to `logger.info()` or `print()`. Grep for `password`, `token`, and `secret` in logging statements and verify none print sensitive values.

- [ ] **Rate limiting on auth endpoints** — `/login`, `/register`, `/refresh`, and `/forgot-password` all have `Depends(rate_limit(...))` in their route decorators. Check `api/v1/endpoints/auth.py`.

- [ ] **Input sanitization on user strings** — All user input goes through Pydantic models with `Field` constraints (`min_length`, `max_length`, `regex`). No raw user strings are interpolated into queries or shell commands. Grep for `subprocess` with `shell=True` — there should be zero matches.
