# comprint-back — Design Specification

## Overview

A self-contained, portable Claude Code plugin for holistic FastAPI backend development. When installed, it provides the `/comprint-back` slash command and an auto-trigger skill that detects backend work. Every coding decision is root-cause driven — zero quick fixes, zero hallucination.

## Core Philosophy

1. **No quick fixes.** Every problem is an investigation. Every fix is a root-cause fix.
2. **No hallucination.** All patterns come from reference files, not from memory. If a reference file has the pattern, use it exactly.
3. **One step at a time.** Each phase completes and verifies before the next begins.
4. **Portable.** Clone and install — works on any machine with zero configuration.

---

## Architecture

### Plugin Structure

```
comprint-back/
├── .claude-plugin/
│   └── plugin.json
├── commands/
│   └── comprint-back.md
├── skills/
│   └── comprint-back-auto/
│       └── SKILL.md
├── references/
│   ├── architecture.md
│   ├── api-design.md
│   ├── database.md
│   ├── security.md
│   ├── exceptions.md
│   ├── logging.md
│   ├── testing.md
│   ├── background-tasks.md
│   └── common-errors.md
├── README.md
└── LICENSE
```

### Component Roles

| Component | Role | When Active |
|-----------|------|-------------|
| `plugin.json` | Registers plugin with Claude Code | On install |
| `commands/comprint-back.md` | The routing brain — classifies tasks, enforces phases, loads references | When user types `/comprint-back` |
| `skills/comprint-back-auto/SKILL.md` | Auto-detects backend work and triggers the same workflow | When Claude sees FastAPI/backend context |
| `references/*.md` | Domain knowledge base — patterns, code examples, rules | Loaded on-demand by the command via Read tool (1-3 files per task, never all 9) |

### Path Resolution

All reference file paths use the `${CLAUDE_PLUGIN_ROOT}` variable, which Claude Code resolves to the plugin's installation directory at runtime. This makes paths portable across machines:

```
${CLAUDE_PLUGIN_ROOT}/references/architecture.md
${CLAUDE_PLUGIN_ROOT}/references/api-design.md
...etc
```

The command body includes explicit Read instructions:
> "After classifying the task, use the Read tool to load the reference files listed in the task classification table. Files are located at `${CLAUDE_PLUGIN_ROOT}/references/`. Read only the files mapped to the classified task type. Never read all 9 files at once."

---

## The Command: `/comprint-back`

### Frontmatter

```yaml
---
description: "Holistic FastAPI backend development — root-cause driven, zero quick fixes. Use for any Python API work: new projects, endpoints, auth, database, testing, debugging, refactoring."
argument-hint: "[describe what you need]"
allowed-tools: ["Read", "Write", "Edit", "Bash", "Glob", "Grep"]
---
```

Note: `Agent` is intentionally excluded from allowed-tools. This is a pure-skills architecture — no agent spawning, no coordination overhead, no failure points. `WebSearch` is also excluded to prevent ungrounded web results from overriding the curated reference patterns.

### Task Classification

When invoked, the command classifies the request before loading any references:

| Task Type | Trigger Signals | References Loaded |
|-----------|----------------|-------------------|
| **Vague / Unclear** | "fix backend", "not working", "build the api" | None yet — enter Phase 0: Clarify |
| **New Project** | "create project", "setup", "init", "new api" | `architecture.md` + `database.md` + `logging.md` |
| **New Endpoint** | "add endpoint", "create route", "new api for..." | `api-design.md` + `exceptions.md` + relevant domain file |
| **Auth Work** | "auth", "login", "jwt", "rbac", "permissions" | `security.md` + `api-design.md` |
| **Database Work** | "model", "migration", "query", "repository" | `database.md` + `architecture.md` |
| **Bug Fix / Debug** | "fix", "error", "broken", "500", "422" | `common-errors.md` + relevant domain file |
| **Testing** | "test", "coverage", "fixture" | `testing.md` + relevant domain file |
| **Background Jobs** | "task", "celery", "queue", "scheduled" | `background-tasks.md` |
| **Logging** | "log", "trace", "monitoring" | `logging.md` |
| **Refactor** | "refactor", "clean up", "restructure" | `architecture.md` + relevant files |

**Resolving "relevant domain file":** Scan the user's request for domain keywords. If the request mentions database/models/queries, the relevant file is `database.md`. If it mentions auth/tokens/permissions, load `security.md`. If multiple domains apply, load up to 3 total reference files maximum, prioritizing the most specific domain to the task.

### Phase Process

Every task goes through phases in order. No skipping.

#### Phase 0: Clarify (vague requests only)

This phase runs ONLY when the request is classified as Vague/Unclear. It runs before the main pipeline.

1. Read the existing codebase — check project structure, recent errors, test output, recent git changes
2. Determine what the actual problem or need is
3. Re-classify the task into one of the specific types in the classification table
4. Proceed to Phase 1 with the clarified classification

Flow: Phase 0 (Clarify) → Phase 1 (Classify, now clear) → Phase 2 → Phase 3 → Phase 4 → Phase 5 → Phase 6

#### Phase 1: Classify
- Parse the user's request
- Match against the task classification table
- If vague/unclear, route to Phase 0 first

#### Phase 2: Load References
- Use the Read tool to load ONLY the reference files identified in Phase 1 (1-3 files max)
- Files are at `${CLAUDE_PLUGIN_ROOT}/references/`
- Read each file completely before proceeding

#### Phase 3: Analyze

**HARD GATE: Do NOT write any code until this phase is complete. Do NOT skip reading the reference files. If a reference file has a pattern for your task, USE IT. Do not invent your own version.**

- Read the existing codebase (relevant files, not everything)
- Understand the current architecture, patterns, and conventions
- For bug fixes: identify the ROOT CAUSE, not the symptom
- For new features: understand how it fits into the existing system

#### Phase 4: Design
- Propose an approach based on reference patterns + existing codebase
- Explain WHY this approach (not just what)
- Present to user for approval before proceeding
- If the approach requires changing existing patterns, explain the trade-offs

**If the user rejects the design:** Return to Phase 3 with the user's feedback as additional context. Re-analyze and propose a revised approach. Repeat until the user approves. Do not proceed to Phase 5 without explicit approval.

#### Phase 5: Implement
- Follow the reference patterns exactly
- Match existing codebase conventions

**The Root-Cause Protocol:**

When encountering ANY problem during implementation:

> 1. **STOP.** Do not write a single line of fix code.
> 2. **WHY does this problem exist?** Trace to origin. Is it a symptom of something deeper?
> 3. **IS a fix even the right move?** Maybe the approach needs rethinking. Maybe the "problem" is correct behavior and the expectation is wrong.
> 4. **What is the HOLISTIC solution?** Not "what makes this error go away" but "what makes the system correct."
> 5. **Present analysis to the user.** Explain what you found, why it exists, what the root solution is.
> 6. **Only then implement.** With understanding, not with a patch.

**IRON LAW: THERE ARE NO QUICK FIXES. THERE ARE NO AUTO-FIXES. EVERY PROBLEM IS AN INVESTIGATION. EVERY FIX IS A ROOT-CAUSE FIX. IF YOU CATCH YOURSELF WRITING A PATCH, STOP AND INVESTIGATE.**

#### Phase 6: Validate

3-level verification (adapted from gsd-verifier):

1. **Exists** — Did you create all the files/functions needed?
2. **Substantive** — Are they real implementations (not stubs, TODOs, or placeholders)?
3. **Wired** — Are they actually connected? Route registered? Dependency injected? Migration created? Tests import the right thing?

Checklist:
- [ ] Response format matches the standard (`success`/`code`/`message`/`data`/`errors`)
- [ ] Error handling follows exceptions.md patterns
- [ ] No blocking calls in async code
- [ ] Layer compliance (no business logic in endpoints, no HTTP in services)
- [ ] Tests cover the new code
- [ ] No hardcoded secrets or credentials

### Anti-Hallucination Mechanisms

**Rationalization Prevention Table:**

| If you're thinking... | The truth is... |
|----------------------|-----------------|
| "I know FastAPI well enough, I don't need the reference file" | The reference file has THIS PROJECT'S patterns. Read it. |
| "This is a simple change, skip the analysis" | Simple changes in the wrong pattern cause cascading fixes. Analyze. |
| "I'll add the tests later" | Phase 6 verification will catch you. Write them now. |
| "This pattern is close enough" | Close enough = inconsistent codebase. Match the reference exactly. |
| "Let me just quickly patch this" | Quick patches are explicitly forbidden. Find the root cause. |
| "I can auto-fix this small bug" | No. Investigate why it exists. Then fix holistically. |

---

## The Auto-Trigger Skill

### Purpose

Catches backend work when the user doesn't explicitly invoke `/comprint-back`. Acts as a detection layer.

### SKILL.md Frontmatter

```yaml
---
name: comprint-back-auto
description: >
  Use when: user is working on FastAPI or Python API code,
  mentions backend/API issues, encounters 422/500/CORS errors,
  asks to "fix the backend" or "build an API", or any request
  that involves Python web endpoints, database models, authentication,
  or API design in a FastAPI project.
---
```

Description contains ONLY triggering conditions (CSO-optimized). No workflow summary — prevents Claude from shortcutting past the full skill body.

### SKILL.md Body Content

The skill body contains:

1. **Verification gate:** Check if this is actually backend/FastAPI work. Look for FastAPI in project dependencies (`pyproject.toml`, `requirements.txt`), or Python API files in the project. If not backend work, exit gracefully.

2. **Context detection:** Determine what the user is working on based on:
   - Open/recently modified files
   - Error messages in the conversation
   - The user's request text

3. **Routing instruction:** "Follow the exact same phase process as the `/comprint-back` command. Read the reference files from `${CLAUDE_PLUGIN_ROOT}/references/` based on the task classification table. Enforce all phases, hard gates, and the root-cause protocol."

4. **The complete phase process** (same as the command body — Phases 0-6, classification table, root-cause protocol, rationalization prevention table). This is duplicated rather than referenced because skills cannot "call" commands.

### Vague Input Handling

| User says | Skill detects | Action |
|-----------|--------------|--------|
| "build the backend" | Backend work, vague scope | Phase 0 Clarify — what backend? What features? |
| "fix the backend" | Backend bug, vague target | Phase 0 Clarify — read error logs, test output, recent changes |
| "something is not working" | Possibly backend if FastAPI project | Check if it's a backend issue, then classify |
| "the API is broken" | API issue | Load `common-errors.md`, enter root-cause protocol |
| "getting a 500 error" | Server error | Load `common-errors.md` + `exceptions.md`, investigate |
| "add a new feature" | Possibly backend if involves API/data | Classify the feature, load relevant references |

---

## Reference Files

### Content Sources

Each reference file is refined from the existing custom_skills/ files in this repo, absorbing the best patterns from:
- The 10 existing SKILL.md files (custom_skills/ + fastapi-coder/)
- The old backend-skill system at ~/.claude/backend-skill/ (Tier 1 patterns only)
- The skillfish fastapi skill's 7 error prevention patterns

### Reference File Specifications

| File | Source Material | Key Content |
|------|----------------|-------------|
| `architecture.md` | fastapi-simple + fastapi-modular | 4-layer pattern (endpoints→services→repositories→models), flat vs modular structures, DI patterns, when to use which |
| `api-design.md` | fastapi-api-design | Response format (`success`/`code`/`message`/`data`/`errors`), endpoint patterns (CRUD), schema design (Base/Create/Update/Response), versioning, TypeScript contract |
| `database.md` | fastapi-database | Async SQLAlchemy setup, Base model + mixins (Timestamp, SoftDelete), BaseAsyncRepository, Alembic migrations, pooling config, query optimization |
| `security.md` | fastapi-security + old security-auditor agent | JWT (access+refresh), bcrypt, RBAC models, auth endpoints, CORS, security headers, rate limiting, audit checklist |
| `exceptions.md` | fastapi-exception | BaseAPIException, concrete types, global handlers, validation error formatting, service→HTTP exception mapping |
| `logging.md` | fastapi-logging | Structured JSON, env config, request ID middleware, ContextVar propagation |
| `testing.md` | fastapi-testing + old test-generator agent | Async fixtures, test DB, AsyncClient, FakeRepository, unit + integration tests, response assertion helpers |
| `background-tasks.md` | fastapi-background-task | BackgroundTasks vs Celery decision table, Celery config, retry logic, Beat scheduling, Docker Compose |
| `common-errors.md` | skillfish fastapi (7 errors) + old backend.md async traps | See detailed list below |

### common-errors.md — Specific Error Patterns to Cover

This file must document prevention patterns for these known issues:

**From skillfish fastapi skill (7 documented errors):**
1. Form data metadata loss — `File()` and `Form()` lose OpenAPI metadata when combined
2. BackgroundTasks overwrite — passing `BackgroundTasks` in multiple dependencies overwrites previous tasks
3. Optional form field regression — `Optional[str] = Form(None)` behavior differences between Pydantic v1/v2
4. `Json` type with `Form` — `Json[dict]` doesn't work with form data as expected
5. `ForwardRef` OpenAPI breakage — forward references in models break OpenAPI schema generation
6. Pydantic v2 union type changes — `Union[A, B]` validation order changed from v1 to v2
7. `ValueError` in validators returning 500 — validators raising `ValueError` produce 500 instead of 422

**From old backend.md async traps:**
8. Blocking calls in async code — using `requests.get()`, `time.sleep()`, or synchronous DB drivers in async endpoints
9. Missing `await` on coroutines — calling async functions without `await` returns a coroutine object, not the result
10. CORS misconfiguration — incorrect origin lists, missing methods/headers, preflight failures

### Word Count Budget

Each reference file should target **1500-2500 words**. If a file exceeds 3000 words, split it or trim non-essential content. The command body should target **2000 words** maximum.

A typical task loads: command body (~2000 words) + 1-3 reference files (~2000 words each) = **~4000-8000 words** of context. This keeps the AI focused and prevents context pollution.

### Progressive Disclosure

- **Level 1 (always in context):** Command description (~50 words) — enough for Claude to know when to route here
- **Level 2 (on invocation):** Command body (~2000 words) — routing logic, phases, rules
- **Level 3 (on demand):** Reference files (~1500-2500 words each) — loaded 1-3 at a time via Read tool based on task classification

### Versioning

Reference files should be reviewed and updated when major dependencies release new versions (FastAPI, SQLAlchemy, Pydantic, etc.). `common-errors.md` is the most likely to need frequent updates as new gotchas are discovered. The plugin manifest version should be bumped when reference content changes significantly.

---

## Distribution

### Installation Method: Direct Git Install

```bash
/plugin install --source github:your-username/comprint-back
```

One command. No marketplace registration. Plugin is cached locally and `/comprint-back` becomes immediately available.

### Plugin Manifest

```json
{
  "name": "comprint-back",
  "description": "Holistic FastAPI backend development — root-cause driven, zero quick fixes",
  "version": "1.0.0",
  "author": {
    "name": "Your Name"
  },
  "category": "development",
  "keywords": ["fastapi", "python", "backend", "api"]
}
```

### What Happens On Install

1. Plugin cached at `~/.claude/plugins/cache/`
2. `/comprint-back` available as slash command immediately
3. Auto-trigger skill detects FastAPI work in background
4. Reference files bundled but only loaded when the command requests them via Read tool
5. Zero configuration from the user

### Skillfish Compatibility

Skillfish compatibility is **unverified** with this directory structure. If needed, a separate `SKILL.md` at the repo root could be added for skillfish discovery, but plugin install is the recommended and tested method.

---

## Design Decisions Log

| Decision | Chosen | Rejected | Why |
|----------|--------|----------|-----|
| Architecture | Pure Skills | Agents, Hybrid | Skills always load, zero failure points. Agent spawning can fail. Maximum portability. |
| Entry point | Command (`/comprint-back`) | Skill-only | Commands give direct slash command. Skills are auto-trigger only. |
| Anti-hallucination | Reference files + hard gates + rationalization tables | Confidence scoring agents | Simpler, no agent overhead, equally effective for skill-based grounding |
| Quick fixes | Forbidden — root-cause protocol | Auto-fix Rule 1 (from gsd-executor) | User's core requirement: every problem is an investigation |
| Distribution | Direct Git Install | Marketplace, skillfish | Simplest for users, no marketplace setup needed |
| Reference loading | On-demand via Read tool (1-3 per task) | Load all, load none | Prevents context pollution while ensuring grounded output |
| Old backend-skill agents | Absorbed into references | Ported as agents | Fewer moving parts. Agent checks become validation rules. |
| Skillfish fastapi skill | Absorbed into common-errors.md | Kept as separate dependency | Single source of truth, no conflicts |
| Project structure skills | Merged into architecture.md | Kept as 2 separate files | Structure choice is an architectural decision, belongs together |
| WebSearch | Excluded from allowed-tools | Included | Ungrounded web results could override curated reference patterns. All needed knowledge lives in references. |
| Agent tool | Excluded from allowed-tools | Included | Pure-skills architecture — no agent spawning, no coordination overhead, no failure points |

---

## Success Criteria

1. **Portability:** `/plugin install --source github:user/comprint-back` works on a fresh machine with zero additional setup
2. **Slash command:** `/comprint-back` appears in `/help` output immediately after install
3. **Auto-detection:** Claude triggers the skill when working on FastAPI code without explicit invocation
4. **No hallucination:** All generated code follows patterns from reference files, not from model memory
5. **No quick fixes:** Every bug fix includes root-cause analysis before any code is written
6. **Context efficiency:** A typical task uses <8000 words of context (command + 1-3 references)
7. **Completeness:** All 9 reference files cover their domain thoroughly with code examples and patterns
8. **Design rejection handling:** User can reject Phase 4 proposals and the system loops back gracefully
