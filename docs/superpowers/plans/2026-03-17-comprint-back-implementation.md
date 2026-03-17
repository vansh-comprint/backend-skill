# comprint-back Plugin Implementation Plan

> **For agentic workers:** REQUIRED: Use superpowers:subagent-driven-development (if subagents available) or superpowers:executing-plans to implement this plan. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a self-contained Claude Code plugin that provides `/comprint-back` for holistic, root-cause-driven FastAPI backend development.

**Architecture:** Pure-skills plugin with one command (routing brain), one auto-trigger skill (detection layer), and 9 reference files (knowledge base). No agents, no external dependencies. Reference files loaded on-demand via Read tool using `${CLAUDE_PLUGIN_ROOT}`.

**Tech Stack:** Claude Code plugin system, Markdown, YAML frontmatter

**Spec:** `docs/superpowers/specs/2026-03-17-comprint-back-design.md`

**Working directory:** All relative paths are relative to the project root — the directory containing `.claude-plugin/`. This is the current working directory (`D:\Cursor codes\backend skill`). Files are created directly here, not in a subdirectory.

---

## File Structure

| File | Responsibility | Source |
|------|---------------|--------|
| `.claude-plugin/plugin.json` | Plugin manifest — registers with Claude Code | New |
| `commands/comprint-back.md` | Slash command — routing brain, phase enforcement, anti-hallucination rules | New (from spec) |
| `skills/comprint-back-auto/SKILL.md` | Auto-trigger skill — detects backend work, same phase process | New (from spec) |
| `references/architecture.md` | 4-layer pattern, project structures, DI | Refined from `custom_skills/fastapi-simple/SKILL.md` + `custom_skills/fastapi-modular/SKILL.md` |
| `references/api-design.md` | Response format, endpoints, schemas, versioning | Refined from `custom_skills/fastapi-api-design/SKILL.md` |
| `references/database.md` | Async SQLAlchemy, repositories, migrations | Refined from `custom_skills/fastapi-database/SKILL.md` |
| `references/security.md` | JWT, RBAC, CORS, rate limiting, audit checklist | Refined from `custom_skills/fastapi-security/SKILL.md` (audit checklist content specified inline below) |
| `references/exceptions.md` | Custom exceptions, global handlers, error mapping | Refined from `custom_skills/fastapi-exception/SKILL.md` |
| `references/logging.md` | Structured JSON logging, request tracing | Refined from `custom_skills/fastapi-logging/SKILL.md` |
| `references/testing.md` | Async fixtures, FakeRepository, integration tests | Refined from `custom_skills/fastapi-testing/SKILL.md` |
| `references/background-tasks.md` | Celery vs BackgroundTasks, scheduling, Docker | Refined from `custom_skills/fastapi-background-task/SKILL.md` |
| `references/common-errors.md` | 10 specific error prevention patterns | New (all content specified inline below — no external source files needed) |
| `README.md` | Installation, usage, what it does | New |
| `LICENSE` | MIT license | New |

---

## Chunk 1: Plugin Foundation

### Task 0: Initialize git repository

- [ ] **Step 1: Initialize git**

```bash
git init
```

- [ ] **Step 2: Create .gitignore**

```
.DS_Store
*.pyc
__pycache__/
```

- [ ] **Step 3: Initial commit**

```bash
git add .gitignore
git commit -m "chore: initialize comprint-back plugin repository"
```

---

### Task 1: Create plugin manifest

**Files:**
- Create: `.claude-plugin/plugin.json`

- [ ] **Step 1: Create the .claude-plugin directory**

```bash
mkdir -p ".claude-plugin"
```

- [ ] **Step 2: Write plugin.json**

```json
{
  "name": "comprint-back",
  "description": "Holistic FastAPI backend development — root-cause driven, zero quick fixes",
  "version": "1.0.0",
  "author": {
    "name": "Comprint"
  },
  "category": "development",
  "keywords": ["fastapi", "python", "backend", "api"]
}
```

- [ ] **Step 3: Commit**

```bash
git add .claude-plugin/plugin.json
git commit -m "feat: add plugin manifest for comprint-back"
```

---

### Task 2: Create the `/comprint-back` command

This is the routing brain — the most critical file in the plugin.

**Files:**
- Create: `commands/comprint-back.md`
- Read first: `docs/superpowers/specs/2026-03-17-comprint-back-design.md` (sections: "The Command", "Anti-Hallucination Mechanisms", "Phase Process")

- [ ] **Step 1: Read the spec for exact content**

Read `docs/superpowers/specs/2026-03-17-comprint-back-design.md`. The command body IS the spec's phase process translated into instructions for Claude. Use these spec sections as your primary source:
- Lines 56-170: The Command section (frontmatter, classification table, all phases, root-cause protocol, rationalization table)
- Lines 43-52: Path resolution with `${CLAUDE_PLUGIN_ROOT}`

- [ ] **Step 2: Create the commands directory**

```bash
mkdir -p commands
```

- [ ] **Step 3: Write the command file**

The command must contain these sections in this order:

1. **YAML frontmatter** with:
   - `description`: "Holistic FastAPI backend development — root-cause driven, zero quick fixes. Use for any Python API work: new projects, endpoints, auth, database, testing, debugging, refactoring."
   - `argument-hint`: "[describe what you need]"
   - `allowed-tools`: ["Read", "Write", "Edit", "Bash", "Glob", "Grep"] — NO Agent, NO WebSearch

2. **Core Philosophy** — 4 rules: no quick fixes, no hallucination, one step at a time, portable

3. **Task Classification Table** — 10 task types with trigger signals and reference file mappings. Include the "relevant domain file" heuristic: scan request for domain keywords, map to specific files, cap at 3 total references.

4. **Reference Loading Instructions** — Explicit: "Use the Read tool to load ONLY the reference files identified in the classification. Files are located at `${CLAUDE_PLUGIN_ROOT}/references/`. Read each file completely before proceeding. Never read all 9 files at once."

5. **Phase 0: Clarify** — For vague/unclear requests only. Steps: read codebase, determine actual need, re-classify, proceed to Phase 1.

6. **Phase 1: Classify** — Parse request, match classification table, route vague to Phase 0.

7. **Phase 2: Load References** — Explicit Read tool instruction with `${CLAUDE_PLUGIN_ROOT}/references/` paths.

8. **Phase 3: Analyze** — HARD GATE in bold (not code fence): **"Do NOT write any code until this phase is complete."** Read existing codebase, understand patterns, identify root causes.

9. **Phase 4: Design** — Propose approach, explain WHY, get user approval. If rejected: return to Phase 3, re-analyze, propose revised. Repeat until approved. Do not proceed without explicit approval.

10. **Phase 5: Implement** — Follow reference patterns exactly. Full Root-Cause Protocol (6 steps: STOP → WHY → IS FIX RIGHT → HOLISTIC SOLUTION → PRESENT → IMPLEMENT). Iron Law in bold.

11. **Phase 6: Validate** — 3-level verification (Exists, Substantive, Wired). 6-item checklist: response format (success/code/message/data/errors), error handling, no blocking async, layer compliance, tests, no hardcoded secrets.

12. **Rationalization Prevention Table** — 6 rows of "If you're thinking... / The truth is..."

**Content guidance:**
- Target ~2000 words (under 2500 max per spec)
- Every instruction must be explicit — assume the reader has zero context
- Reference `${CLAUDE_PLUGIN_ROOT}/references/` for all file paths
- No marketing language, no fluff

- [ ] **Step 4: Verify the command file has correct frontmatter**

```bash
head -10 commands/comprint-back.md
```

Expected: YAML frontmatter with `description`, `argument-hint`, `allowed-tools`

- [ ] **Step 5: Verify word count is within budget**

```bash
wc -w commands/comprint-back.md
```

Expected: ~1500-2000 words (under 2500 max)

- [ ] **Step 6: Verify ${CLAUDE_PLUGIN_ROOT} references are present**

```bash
grep "CLAUDE_PLUGIN_ROOT" commands/comprint-back.md
```

Expected: At least one reference to `${CLAUDE_PLUGIN_ROOT}/references/`

- [ ] **Step 7: Commit**

```bash
git add commands/comprint-back.md
git commit -m "feat: add /comprint-back command — routing brain with 7-phase process"
```

---

### Task 3: Create the auto-trigger skill

**Files:**
- Create: `skills/comprint-back-auto/SKILL.md`
- Read first: `docs/superpowers/specs/2026-03-17-comprint-back-design.md` (lines 190-237: Auto-Trigger Skill section)
- Read first: `commands/comprint-back.md` (already created — the phase process to duplicate)

- [ ] **Step 1: Read the spec and command file**

Read the spec's Auto-Trigger Skill section for the frontmatter and behavior spec. Then read `commands/comprint-back.md` for the exact phase process content to duplicate.

- [ ] **Step 2: Create the skills directory**

```bash
mkdir -p skills/comprint-back-auto
```

- [ ] **Step 3: Write the SKILL.md file**

The skill must contain:

1. **YAML frontmatter:**
   - `name: comprint-back-auto`
   - `description:` CSO-optimized — ONLY triggering conditions, no workflow summary. Use the exact description from the spec: "Use when: user is working on FastAPI or Python API code, mentions backend/API issues, encounters 422/500/CORS errors, asks to 'fix the backend' or 'build an API', or any request that involves Python web endpoints, database models, authentication, or API design in a FastAPI project."

2. **Verification gate:** Check if this is actually backend/FastAPI work. Look for FastAPI in project dependencies (pyproject.toml, requirements.txt). If not backend work, exit gracefully.

3. **Context detection:** Determine what the user is working on based on open/recently modified files, error messages, request text.

4. **The complete phase process** — COPY the entire Phase 0-6 process from `commands/comprint-back.md`. Include: task classification table, reference loading instructions (same `${CLAUDE_PLUGIN_ROOT}/references/` paths), root-cause protocol, iron law, rationalization prevention table, 3-level verification.

   This duplication is necessary because skills cannot invoke commands. Both files must be independently complete.

5. **Vague input handling table** — 6 rows mapping vague user statements to actions (from the spec).

- [ ] **Step 4: Verify frontmatter is CSO-optimized**

Read the description field. It must contain ONLY triggering conditions. If it contains workflow steps (e.g., "analyzes code, loads references, follows phases"), it will cause Claude to shortcut past the skill body. Fix it.

- [ ] **Step 5: Verify phase process matches the command file**

Compare the phase process sections between `commands/comprint-back.md` and `skills/comprint-back-auto/SKILL.md`. The core content must be identical:
- Task classification table
- Phase 0-6 instructions
- Root-cause protocol (6 steps)
- Iron law
- Rationalization prevention table (6 rows)
- 3-level verification + checklist

Any differences in these sections are bugs. Fix them.

- [ ] **Step 6: Verify word count**

```bash
wc -w skills/comprint-back-auto/SKILL.md
```

Expected: ~2500-3500 words (longer than command due to verification gate + context detection + vague input table)

- [ ] **Step 7: Commit**

```bash
git add skills/comprint-back-auto/SKILL.md
git commit -m "feat: add auto-trigger skill for backend work detection"
```

---

### Task 4: Verify plugin foundation works

- [ ] **Step 1: Check plugin structure is correct**

```bash
ls -la .claude-plugin/
ls -la commands/
ls -la skills/comprint-back-auto/
```

Expected: plugin.json, comprint-back.md, SKILL.md all exist

- [ ] **Step 2: Verify plugin.json is valid JSON**

```bash
python -c "import json; json.load(open('.claude-plugin/plugin.json')); print('Valid JSON')"
```

Expected: "Valid JSON"

- [ ] **Step 3: Verify YAML frontmatter in command and skill files**

Check that both files start with `---` and end the frontmatter with `---`. Verify `description`, `argument-hint` (command), `name` (skill) are present.

- [ ] **Step 4: Verify ${CLAUDE_PLUGIN_ROOT} references are consistent**

```bash
grep -r "CLAUDE_PLUGIN_ROOT" commands/ skills/
```

Expected: Both files reference `${CLAUDE_PLUGIN_ROOT}/references/` for loading reference files.

---

## Chunk 2: Reference Files — Architecture & API Design

### Task 5: Create references/architecture.md

**Files:**
- Create: `references/architecture.md`
- Read: `custom_skills/fastapi-simple/SKILL.md` (source)
- Read: `custom_skills/fastapi-modular/SKILL.md` (source)

- [ ] **Step 1: Create the references directory**

```bash
mkdir -p references
```

- [ ] **Step 2: Read both source files**

Read `custom_skills/fastapi-simple/SKILL.md` and `custom_skills/fastapi-modular/SKILL.md` completely.

- [ ] **Step 3: Write references/architecture.md**

Extract and restructure into these sections:

1. **4-Layer Architecture** — endpoints → services → repositories → models. Rules for each layer (what belongs, what doesn't). Extract from fastapi-simple's architecture section.

2. **When to Use Simple vs Modular**
   - Simple: <50 endpoints, 1-4 devs, single domain, MVP/prototype
   - Modular: 50+ endpoints, 4+ devs, multiple domains, enterprise

3. **Simple Project Structure** — flat directory layout from fastapi-simple. Include the actual directory tree.

4. **Modular Project Structure** — domain-based layout from fastapi-modular. Include module anatomy.

5. **Dependency Injection Patterns** — `Depends()`, Annotated type aliases, service injection. Extract from both sources.

6. **Inter-Module Communication** (modular only) — Event bus, Protocols, API composition. Extract from fastapi-modular.

**Refinement rules:**
- Strip YAML frontmatter (these are reference files, not skills)
- Remove any "when to use this skill" language
- Remove redundant explanations — keep patterns and code examples
- Target 1500-2500 words
- No marketing language

- [ ] **Step 4: Verify word count**

```bash
wc -w references/architecture.md
```

Expected: 1500-2500 words (flag if over 3000 — trim)

- [ ] **Step 5: Commit**

```bash
git add references/architecture.md
git commit -m "feat: add architecture reference — 4-layer pattern, simple + modular structures"
```

---

### Task 6: Create references/api-design.md

**Files:**
- Create: `references/api-design.md`
- Read: `custom_skills/fastapi-api-design/SKILL.md` (source)

- [ ] **Step 1: Read the source file**

Read `custom_skills/fastapi-api-design/SKILL.md` completely.

- [ ] **Step 2: Write references/api-design.md**

Extract and restructure into these sections:

1. **Standard Response Format** — the 5-field format: `success` (bool), `code` (int), `message` (str), `data` (any), `errors` (list|null). Include the StandardResponse and PaginatedResponse Pydantic models. Include error response examples.

2. **Endpoint Patterns** — Create, Read, List (with pagination), Update, Delete. Include complete route function examples for each.

3. **Schema Design** — Base/Create/Update/Response pattern. Show how to compose schemas. Include Pydantic v2 `ConfigDict(from_attributes=True)`.

4. **Annotated Dependency Injection** — Type aliases for clean signatures (`CurrentUser`, `DBSession`, `UserService`).

5. **API Versioning** — URL path approach (/api/v1/, /api/v2/), deprecation middleware with Sunset headers. Brief — only if the project needs it.

6. **TypeScript Frontend Contract** — Show how the response format maps to TypeScript interfaces for frontend devs.

**Refinement rules:**
- Ensure the response format uses all 5 fields consistently (success/code/message/data/errors)
- Target 1500-2500 words

- [ ] **Step 3: Verify response format consistency**

```bash
grep -c "success" references/api-design.md
grep -c "errors" references/api-design.md
```

Expected: Both fields appear multiple times — they're part of every response example.

- [ ] **Step 4: Verify word count**

```bash
wc -w references/api-design.md
```

Expected: 1500-2500 words

- [ ] **Step 5: Commit**

```bash
git add references/api-design.md
git commit -m "feat: add api-design reference — response format, endpoint patterns, schemas"
```

---

## Chunk 3: Reference Files — Data Layer

### Task 7: Create references/database.md

**Files:**
- Create: `references/database.md`
- Read: `custom_skills/fastapi-database/SKILL.md` (source)

- [ ] **Step 1: Read the source file**

Read `custom_skills/fastapi-database/SKILL.md` completely.

- [ ] **Step 2: Write references/database.md**

Extract and restructure into these sections:

1. **Async Engine & Session Setup** — PostgreSQL + AsyncPG driver, `create_async_engine`, `async_sessionmaker`, `get_db_session` dependency. Include connection string format and pool config (`pool_size=20`, `max_overflow=10`).

2. **Base Model + Mixins** — `TimestampMixin` (created_at, updated_at), `SoftDeleteMixin` (deleted_at), `Base` class.

3. **BaseAsyncRepository** — Generic CRUD repository with `get`, `get_all`, `create`, `update`, `delete` methods. Type-safe with generics.

4. **Concrete Repository Example** — `UserRepository` extending BaseAsyncRepository with custom query methods.

5. **Alembic Async Migrations** — Configuration for async, timestamp-based naming (`YYYYMMDD_HHMMSS_description.py`), env.py setup.

6. **Query Optimization** — N+1 prevention with `selectinload`, bulk operations, when to use `joinedload` vs `selectinload`.

7. **Model Relationships** — One-to-many, many-to-many with association tables. Lazy loading configuration.

**Refinement rules:**
- All code must use async patterns (AsyncSession, await)
- Target 1500-2500 words

- [ ] **Step 3: Verify all code is async**

```bash
grep -c "async" references/database.md
grep -c "await" references/database.md
```

Expected: Multiple occurrences of both — no sync patterns.

- [ ] **Step 4: Verify word count**

```bash
wc -w references/database.md
```

Expected: 1500-2500 words

- [ ] **Step 5: Commit**

```bash
git add references/database.md
git commit -m "feat: add database reference — async SQLAlchemy, repositories, migrations"
```

---

### Task 8: Create references/exceptions.md

**Files:**
- Create: `references/exceptions.md`
- Read: `custom_skills/fastapi-exception/SKILL.md` (source)

- [ ] **Step 1: Read the source file**

Read `custom_skills/fastapi-exception/SKILL.md` completely.

- [ ] **Step 2: Write references/exceptions.md**

Extract and restructure into these sections:

1. **BaseAPIException** — Base class with `status_code`, `code`, `message`, `errors` fields.

2. **Concrete Exception Types** — NotFoundError, UnauthorizedError, ForbiddenError, ConflictError, BadRequestError, RateLimitError. Each with default status code and message.

3. **Global Exception Handlers** — Three handlers: BaseAPIException handler, RequestValidationError handler (clean field names — strip "body." prefix), unhandled Exception handler (500, never leak stack traces).

4. **Validation Error Formatting** — Transform FastAPI's default validation errors into the standard response format.

5. **Service → HTTP Exception Mapping** — Services raise domain exceptions, global handler maps to HTTP responses. Services never import HTTP concepts.

6. **Error Logging with Request Context** — Log errors with request ID, method, path, user ID.

**Refinement rules:**
- Every error response must match the standard format (success/code/message/data/errors)
- Services raise domain exceptions, NOT HTTPException
- Target 1500-2500 words

- [ ] **Step 3: Verify word count**

```bash
wc -w references/exceptions.md
```

Expected: 1500-2500 words

- [ ] **Step 4: Commit**

```bash
git add references/exceptions.md
git commit -m "feat: add exceptions reference — error handling, global handlers, FE-friendly errors"
```

---

## Chunk 4: Reference Files — Security

### Task 9: Create references/security.md

**Files:**
- Create: `references/security.md`
- Read: `custom_skills/fastapi-security/SKILL.md` (source)

- [ ] **Step 1: Read the source file**

Read `custom_skills/fastapi-security/SKILL.md` completely.

- [ ] **Step 2: Write references/security.md**

Extract and restructure into these sections:

1. **JWT Token Creation & Verification** — Access token (short-lived, 15-30 min) + refresh token (long-lived, 7 days). `create_access_token()`, `create_refresh_token()`, `verify_token()`. python-jose or PyJWT.

2. **Password Hashing** — bcrypt via passlib. `hash_password()`, `verify_password()`.

3. **RBAC Models** — User → Roles → Permissions (many-to-many). Association tables. `has_role()`, `has_permission()`.

4. **Auth Dependencies** — Annotated types: `CurrentUser`, `AdminUser`.

5. **Auth Endpoints** — Register, login (returns access + refresh tokens), refresh.

6. **CORS Configuration** — `CORSMiddleware` setup. Explicit origin lists (never `*` in production).

7. **Security Headers Middleware** — X-Content-Type-Options, X-Frame-Options, Strict-Transport-Security.

8. **Rate Limiting** — Redis-based rate limiter. Per-endpoint configuration.

9. **Security Audit Checklist** — concrete, grep-able checks (no external source file needed — content specified here):
   - [ ] JWT has `exp` claim and is verified
   - [ ] Passwords hashed with bcrypt (not MD5/SHA)
   - [ ] No hardcoded secrets (grep for `SECRET`, `PASSWORD`, `API_KEY` in code)
   - [ ] SQL queries use parameterized statements (no f-strings)
   - [ ] CORS origins are explicit (no wildcard in production)
   - [ ] Sensitive data not logged (passwords, tokens, PII)
   - [ ] Rate limiting on auth endpoints
   - [ ] Input sanitization on user-provided strings

**Refinement rules:**
- All auth code must use async
- Include complete code examples (not pseudocode)
- Target 2000-2500 words (largest reference file)

- [ ] **Step 3: Verify word count**

```bash
wc -w references/security.md
```

Expected: 2000-2500 words (under 3000 max)

- [ ] **Step 4: Commit**

```bash
git add references/security.md
git commit -m "feat: add security reference — JWT, RBAC, CORS, audit checklist"
```

---

## Chunk 5: Reference Files — Operations

### Task 10: Create references/logging.md

**Files:**
- Create: `references/logging.md`
- Read: `custom_skills/fastapi-logging/SKILL.md` (source)

- [ ] **Step 1: Read the source file**

Read `custom_skills/fastapi-logging/SKILL.md` completely.

- [ ] **Step 2: Write references/logging.md**

Extract and restructure into:

1. **Logger Setup** — JSON formatting with `python-json-logger`. Environment-specific levels (DEBUG/INFO/WARNING).
2. **Request ID Middleware** — UUID generation per request. `ContextVar` for propagation.
3. **Request/Response Logging Middleware** — Log method, path, status, duration on every request.
4. **Service-Level Logging** — How to log from services with request context. `get_logger()` helper.
5. **JSON Output Compatibility** — Compatible with ELK, Datadog, CloudWatch, Grafana Loki.

Target: 1500-2000 words (smallest reference).

- [ ] **Step 3: Verify word count**

```bash
wc -w references/logging.md
```

Expected: 1500-2000 words

- [ ] **Step 4: Commit**

```bash
git add references/logging.md
git commit -m "feat: add logging reference — structured JSON, request tracing"
```

---

### Task 11: Create references/testing.md

**Files:**
- Create: `references/testing.md`
- Read: `custom_skills/fastapi-testing/SKILL.md` (source)

- [ ] **Step 1: Read the source file**

Read `custom_skills/fastapi-testing/SKILL.md` completely.

- [ ] **Step 2: Write references/testing.md**

Extract and restructure into:

1. **Test Structure** — Directory layout for simple and modular projects.
2. **Core conftest.py** — Async fixtures: `db_engine`, `db_session`, `client` (AsyncClient with ASGITransport), `test_user`.
3. **Test Database Setup** — SQLite + aiosqlite for fast test DB. Dependency override pattern.
4. **FakeRepository Pattern** — Protocol-based repositories for unit testing without mocking. Complete FakeUserRepository example.
5. **Unit Tests** — Using FakeRepository. Arrange-Act-Assert pattern.
6. **Integration Tests** — Using AsyncClient. Validating the full 5-field response format.
7. **Response Format Assertion Helpers** — `assert_success_response()`, `assert_error_response()`.
8. **pytest Configuration** — `asyncio_mode = "auto"`, coverage with `fail_under=80`.

Target: 2000-2500 words.

- [ ] **Step 3: Verify word count**

```bash
wc -w references/testing.md
```

Expected: 2000-2500 words

- [ ] **Step 4: Commit**

```bash
git add references/testing.md
git commit -m "feat: add testing reference — async fixtures, FakeRepository, integration tests"
```

---

### Task 12: Create references/background-tasks.md

**Files:**
- Create: `references/background-tasks.md`
- Read: `custom_skills/fastapi-background-task/SKILL.md` (source)

- [ ] **Step 1: Read the source file**

Read `custom_skills/fastapi-background-task/SKILL.md` completely.

- [ ] **Step 2: Write references/background-tasks.md**

Extract and restructure into:

1. **Decision Table** — When to use FastAPI BackgroundTasks vs Celery:
   - BackgroundTasks: lightweight, in-process, <30 seconds, no retry needed
   - Celery: heavy, distributed, any duration, retry/scheduling needed
2. **BackgroundTasks Usage** — `BackgroundTasks` dependency, adding tasks, common patterns.
3. **Celery Configuration** — Broker (Redis), backend, serialization, retry settings.
4. **Task Definitions** — `@celery_app.task` with retry logic.
5. **Task Status Endpoints** — Long-running job status API.
6. **Celery Beat** — Periodic/scheduled tasks.
7. **Docker Compose** — Worker + Beat services configuration.

Target: 1500-2500 words.

- [ ] **Step 3: Verify word count**

```bash
wc -w references/background-tasks.md
```

Expected: 1500-2500 words

- [ ] **Step 4: Commit**

```bash
git add references/background-tasks.md
git commit -m "feat: add background-tasks reference — Celery vs BackgroundTasks, scheduling"
```

---

## Chunk 6: Common Errors, README & Final Verification

### Task 13: Create references/common-errors.md

This file is NEW — all content is specified inline below. No external source files need to be read.

**Files:**
- Create: `references/common-errors.md`

- [ ] **Step 1: Write references/common-errors.md**

Document prevention patterns for these 10 specific issues. Use this format for each error:

```markdown
## N. Error Name

**Symptom:** What the developer sees
**Root Cause:** Why it happens
**Prevention:** How to avoid it (with code example)
**Detection:** How to find this in existing code (grep pattern if applicable)
```

**The 10 errors:**

1. **Form data metadata loss** — `File()` and `Form()` lose OpenAPI metadata when combined in the same endpoint. Prevention: use separate models or document the limitation.

2. **BackgroundTasks overwrite** — passing `BackgroundTasks` in multiple dependencies overwrites previous tasks. Prevention: pass BackgroundTasks only in the endpoint function, not in dependencies.

3. **Optional form field regression** — `Optional[str] = Form(None)` behavior differs between Pydantic v1/v2. Prevention: always use `str | None = Form(default=None)` with explicit default.

4. **Json type with Form** — `Json[dict]` doesn't work with form data as expected. Prevention: accept as `str` and parse manually, or use request body instead of form.

5. **ForwardRef OpenAPI breakage** — forward references in models break OpenAPI schema generation. Prevention: use `model_rebuild()` after all models are defined, or avoid forward refs.

6. **Pydantic v2 union type changes** — `Union[A, B]` validation order changed from v1 to v2 (v2 uses left-to-right strict). Prevention: order union types from most specific to least specific.

7. **ValueError in validators returning 500** — validators raising `ValueError` produce 500 instead of 422. Prevention: always raise `ValueError` inside `@field_validator`, never outside Pydantic context.

8. **Blocking calls in async code** — using `requests.get()`, `time.sleep()`, or synchronous DB drivers in async endpoints blocks the event loop. Prevention: use `httpx.AsyncClient`, `asyncio.sleep()`, and async DB drivers.

9. **Missing await on coroutines** — calling async function without `await` returns a coroutine object, not the result. Prevention: always `await` async calls. Use `ruff` to catch unawaited coroutines.

10. **CORS misconfiguration** — incorrect origin lists, missing methods/headers, preflight failures. Prevention: explicit origin list with exact URLs including port.

Target: 2000-2500 words.

- [ ] **Step 2: Verify all 10 errors are covered**

```bash
grep -c "^## " references/common-errors.md
```

Expected: 10 (one heading per error pattern)

- [ ] **Step 3: Verify word count**

```bash
wc -w references/common-errors.md
```

Expected: 2000-2500 words

- [ ] **Step 4: Commit**

```bash
git add references/common-errors.md
git commit -m "feat: add common-errors reference — 10 error prevention patterns"
```

---

### Task 14: Create README.md

**Files:**
- Create: `README.md`

- [ ] **Step 1: Write README.md**

Sections:

1. **Title:** comprint-back
2. **One-line description:** Holistic FastAPI backend development for Claude Code — root-cause driven, zero quick fixes.
3. **Installation:**
   ```bash
   /plugin install --source github:YOUR-USERNAME/comprint-back
   ```
4. **Usage:**
   - Type `/comprint-back [describe what you need]` in Claude Code
   - Or just start working on FastAPI code — the auto-trigger skill detects it
5. **What it does:**
   - Routes your request to the right domain knowledge (9 reference areas)
   - Enforces a 7-phase process: Clarify → Classify → Load → Analyze → Design → Implement → Validate
   - Every problem gets root-cause analysis before any code is written
   - All patterns come from curated reference files, not model memory
6. **Reference areas:** List the 9 reference files with one-line descriptions
7. **Philosophy:** Brief version of the 4 core principles

Keep it concise — under 150 lines.

- [ ] **Step 2: Commit**

```bash
git add README.md
git commit -m "docs: add README with installation and usage instructions"
```

---

### Task 15: Create LICENSE

**Files:**
- Create: `LICENSE`

- [ ] **Step 1: Write MIT LICENSE file**

Standard MIT license with current year (2026) and author name.

- [ ] **Step 2: Commit**

```bash
git add LICENSE
git commit -m "docs: add MIT license"
```

---

### Task 16: End-to-end verification

- [ ] **Step 1: Verify complete plugin file structure**

```bash
ls -R .claude-plugin/ commands/ skills/ references/
```

Expected — all 14 plugin files present:
- `.claude-plugin/plugin.json`
- `commands/comprint-back.md`
- `skills/comprint-back-auto/SKILL.md`
- `references/architecture.md`
- `references/api-design.md`
- `references/database.md`
- `references/security.md`
- `references/exceptions.md`
- `references/logging.md`
- `references/testing.md`
- `references/background-tasks.md`
- `references/common-errors.md`
- `README.md`
- `LICENSE`

- [ ] **Step 2: Verify all reference files are within word budget**

```bash
for f in references/*.md; do echo "$f: $(wc -w < "$f") words"; done
```

Expected: Each file between 1500-2500 words. Flag any over 3000 for trimming.

- [ ] **Step 3: Verify ${CLAUDE_PLUGIN_ROOT} usage is consistent**

```bash
grep -r "CLAUDE_PLUGIN_ROOT" commands/ skills/
```

Expected: Both command and skill reference `${CLAUDE_PLUGIN_ROOT}/references/`

- [ ] **Step 4: Verify no hallucination vectors**

Check that:
- `allowed-tools` in command does NOT include `Agent` or `WebSearch`
- Skill description contains ONLY triggering conditions (no workflow steps)
- All code examples in references use async patterns
- Response format is consistently 5 fields (success/code/message/data/errors) everywhere

- [ ] **Step 5: Verify plugin.json is valid**

```bash
python -c "import json; json.load(open('.claude-plugin/plugin.json')); print('Valid JSON')"
```

- [ ] **Step 6: Verify phase process consistency between command and skill**

Compare the core phase content between `commands/comprint-back.md` and `skills/comprint-back-auto/SKILL.md`. The classification table, phases 0-6, root-cause protocol, iron law, and rationalization table must match.

- [ ] **Step 7: Clean up source files (USER DECISION REQUIRED)**

The `custom_skills/` and `fastapi-coder/` directories are source material that has been absorbed into `references/`. They can be removed from the plugin to avoid confusion.

**IMPORTANT:** Before deleting, verify:
1. Git has all changes committed (`git status` shows clean)
2. All reference files are complete and correct (Steps 1-6 above pass)
3. The user explicitly approves this deletion

The `docs/` directory is NOT part of the distributed plugin — it contains development artifacts (spec and plan). It should be kept for reference but can be excluded from distribution via `.gitignore` if desired.

```bash
# Only after user approval:
rm -rf custom_skills/ fastapi-coder/
git add -A
git commit -m "chore: remove source skill files — content absorbed into references/"
```

- [ ] **Step 8: Final verification**

```bash
git log --oneline
```

Expected: Clean commit history showing the plugin built incrementally, one component at a time.
