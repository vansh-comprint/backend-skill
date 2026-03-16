# comprint-back

Holistic FastAPI backend development for Claude Code — root-cause driven, zero quick fixes.

## Installation

```bash
/plugin install --source github:YOUR-USERNAME/comprint-back
```

## Usage

**Explicit invocation:**
```
/comprint-back add user authentication with JWT and role-based access
```

**Auto-detection:** Just start working on FastAPI code — the plugin detects it and activates automatically.

## What It Does

- **Routes** your request to the right domain knowledge (9 reference areas)
- **Enforces** a 7-phase process: Clarify, Classify, Load, Analyze, Design, Implement, Validate
- **Investigates** every problem for root cause before writing any code
- **Grounds** all patterns in curated reference files, not model memory

## Reference Areas

| Reference | Covers |
|-----------|--------|
| `architecture.md` | 4-layer pattern, simple + modular project structures, DI |
| `api-design.md` | Response format, endpoint patterns, schemas, versioning |
| `database.md` | Async SQLAlchemy, repositories, migrations, optimization |
| `security.md` | JWT, RBAC, CORS, rate limiting, security audit checklist |
| `exceptions.md` | Custom exceptions, global handlers, FE-friendly errors |
| `logging.md` | Structured JSON logging, request tracing |
| `testing.md` | Async fixtures, FakeRepository, integration tests |
| `background-tasks.md` | Celery vs BackgroundTasks, scheduling, Docker |
| `common-errors.md` | 10 documented error prevention patterns |

## Philosophy

1. **No quick fixes.** Every problem is an investigation. Every fix is a root-cause fix.
2. **No hallucination.** All patterns come from reference files, not from memory.
3. **One step at a time.** Each phase completes and verifies before the next begins.
4. **Portable.** Clone and install — works on any machine with zero configuration.
