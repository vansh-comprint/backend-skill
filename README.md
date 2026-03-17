# comprint-back

A Claude Code plugin that gives you a complete FastAPI development system. One command covers everything — project scaffolding, endpoints, authentication, database, testing, error handling, background jobs, and more.

Built on **FastAPI + Pydantic v2 + SQLAlchemy 2.0 async + PostgreSQL** with production-ready patterns.

## Installation

```bash
/plugin install --source github:vansh-comprint/backend-skill
```

That's it. `/comprint-back` is immediately available.

## Usage

```
/comprint-back scaffold a new FastAPI project with auth and database
/comprint-back add a CRUD endpoint for products
/comprint-back set up JWT authentication with role-based access control
/comprint-back fix this 422 validation error
/comprint-back add Celery background tasks with Redis
```

Or just start working on FastAPI code — the plugin auto-detects backend work and activates.

## What You Get

### 9 Domain Knowledge Areas

The plugin carries a curated knowledge base covering the full FastAPI stack. When you ask for something, it loads only the relevant references — keeping context focused.

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

### Smart Routing

Tell it what you need. It figures out which knowledge areas apply and loads them.

- "add auth" → loads Security + API Design
- "create a new project" → loads Architecture + Database + Logging
- "fix this 500 error" → loads Common Errors + relevant domain
- "something's not working" → investigates first, then loads what's needed

### How It Works

Every task follows a structured process:

1. **Classify** — What kind of work is this?
2. **Load** — Pull in only the relevant reference files
3. **Analyze** — Read the existing codebase, understand current patterns
4. **Design** — Propose an approach, get your approval
5. **Implement** — Follow the reference patterns exactly
6. **Validate** — Verify everything is wired up, no stubs, no gaps

## Tech Stack

- Python 3.12+
- FastAPI
- Pydantic v2
- SQLAlchemy 2.0 (async)
- PostgreSQL + AsyncPG
- Alembic
- pytest + pytest-asyncio + httpx
- Celery + Redis
- python-jose (JWT)
- passlib + bcrypt

## Project Structures

The plugin supports two project layouts and helps you pick the right one:

**Simple** — flat structure for small-to-medium APIs (<50 endpoints, 1-4 devs)

**Modular** — domain-based modules for larger systems (50+ endpoints, multiple teams)

## License

MIT
