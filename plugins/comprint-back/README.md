# comprint-back

A Claude Code plugin that gives you a complete FastAPI development system. One command covers everything — project scaffolding, endpoints, authentication, database, testing, error handling, background jobs, and more.

Built on **FastAPI + Pydantic v2 + SQLAlchemy 2.0 async + PostgreSQL** with production-ready patterns.

## Usage

```
/comprint-back scaffold a new FastAPI project with auth and database
/comprint-back add a CRUD endpoint for products
/comprint-back set up JWT authentication with role-based access control
/comprint-back fix this 422 validation error
/comprint-back add Celery background tasks with Redis
```

Or just start working on FastAPI code — the plugin auto-detects backend work and activates.

## 9 Domain Knowledge Areas

| Domain | What's Inside |
|--------|--------------|
| **Architecture** | 4-layer pattern (endpoints → services → repositories → models), flat + modular project structures, dependency injection |
| **API Design** | Standardized response format, CRUD endpoint patterns, Pydantic schema design, API versioning, TypeScript frontend contracts |
| **Database** | Async SQLAlchemy 2.0, generic repository pattern, Alembic migrations, query optimization, relationship patterns |
| **Security** | JWT (access + refresh tokens), RBAC with permissions, CORS, rate limiting, security headers, audit checklist |
| **Error Handling** | Custom exception hierarchy, global handlers, validation error formatting, FE-friendly error responses |
| **Logging** | Structured JSON logging, request ID tracing, per-request context propagation |
| **Testing** | Async fixtures, FakeRepository pattern (no mocking needed), integration tests, coverage config |
| **Background Tasks** | FastAPI BackgroundTasks vs Celery decision guide, task scheduling, Docker Compose for workers |
| **Common Errors** | 10 documented prevention patterns — Pydantic v2 gotchas, async traps, CORS issues, form data quirks |

## Smart Routing

Tell it what you need. It figures out which knowledge areas apply and loads them.

- "add auth" → loads Security + API Design
- "create a new project" → loads Architecture + Database + Logging
- "fix this 500 error" → loads Common Errors + relevant domain
- "something's not working" → investigates first, then loads what's needed

## Tech Stack

Python 3.12+ · FastAPI · Pydantic v2 · SQLAlchemy 2.0 (async) · PostgreSQL + AsyncPG · Alembic · pytest + pytest-asyncio + httpx · Celery + Redis · python-jose (JWT) · passlib + bcrypt
