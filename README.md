# Backend Fortress

FastAPI backend development skill for Claude Code. Enforces proper architecture, prevents hallucination, includes all production patterns.

## What's Inside

| File | Purpose |
|------|---------|
| `skills/backend.md` | Main skill - 2400+ lines of patterns, code templates, anti-hallucination rules |
| `agents/` | Specialized agents (security, architecture, code review, database, testing) |
| `hooks/pre-write-validation.md` | Validates code before writing |
| `checklists/endpoint-checklist.md` | Endpoint completion checklist |

## Patterns Included

- **Architecture**: Enhanced MVC + Service Layer (Endpoint → Service → Repository → Model)
- **Auth**: JWT with refresh tokens, RBAC with multiple roles/permissions
- **Database**: Async SQLAlchemy 2.0, Alembic migrations, soft delete, relationships
- **API**: Pagination, versioning (v1/v2), standardized responses `{code, data, message}`
- **Background Jobs**: Celery with Redis, scheduled tasks, retries
- **Security**: Rate limiting, input validation, CORS, security headers
- **Caching**: Redis caching with decorator pattern
- **Logging**: Structured JSON logging with request tracking
- **Testing**: Async fixtures, API tests

## Installation

Copy to your project's `.claude/plugins/` folder:

```bash
# Clone
git clone git@github.com:comprint-dev/backend-skill.git

# Copy to your project
cp -r backend-skill /path/to/your/project/.claude/plugins/
```

Or add directly to an existing plugin's structure.

## Usage

Once installed, the skill triggers on keywords like:
- `backend`, `fastapi`, `api`, `endpoint`, `crud`
- `authentication`, `database`, `service layer`

Or invoke directly with `/backend`.

## File Naming Convention

| Component | File | Class |
|-----------|------|-------|
| Model | `User.py` | `class User` |
| Schema | `UserSchema.py` | `UserCreate`, `UserUpdate`, `UserResponse` |
| Repository | `UserRepository.py` | `class UserRepository` |
| Service | `UserService.py` | `class UserService` |

## Response Format

All endpoints return:
```json
{
  "code": 200,
  "data": { ... },
  "message": "Success"
}
```

## Requirements

For projects using this skill:
```
fastapi>=0.128.0
sqlalchemy[asyncio]>=2.0.30
asyncpg>=0.29.0
pydantic>=2.11.0
pydantic-settings>=2.5.0
python-jose[cryptography]>=3.3.0
passlib[bcrypt]>=1.7.4
celery[redis]>=5.3.0
redis>=5.0.0
alembic>=1.13.0
```

## License

MIT
