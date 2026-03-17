---
description: "Holistic FastAPI backend development — root-cause driven, zero quick fixes. Use for any Python API work: new projects, endpoints, auth, database, testing, debugging, refactoring."
argument-hint: "[describe what you need]"
allowed-tools: ["Read", "Write", "Edit", "Bash", "Glob", "Grep"]
---

# /comprint-back

## Core Philosophy

Every action you take under this command follows four rules. No exceptions.

1. **No quick fixes.** Every problem is an investigation. Every fix addresses the root cause. If you catch yourself writing a patch to make an error disappear, stop and investigate why the error exists.

2. **No hallucination.** You do not rely on your training data for project-specific patterns. You read the reference files provided by this plugin and follow them exactly. If a reference file shows a pattern for your task, you use that pattern — you do not invent your own version.

3. **One step at a time.** You follow the phase process below in order. You do not skip phases. You do not combine phases. You do not jump to implementation before analysis and design are complete.

4. **Follow reference patterns exactly.** The reference files contain this project's conventions — response formats, error handling, layer architecture, testing patterns. Match them precisely. "Close enough" is not acceptable because it creates inconsistency.

---

## Task Classification Table

When invoked, classify the user's request before loading any reference files.

| Task Type | Trigger Signals | References to Load |
|-----------|----------------|-------------------|
| **Vague / Unclear** | "fix backend", "not working", "build the api", ambiguous requests | None yet — enter Phase 0: Clarify first |
| **New Project** | "create project", "setup", "init", "new api", "start from scratch" | `architecture.md` + `database.md` + `logging.md` |
| **New Endpoint** | "add endpoint", "create route", "new api for...", "add a GET/POST" | `api-design.md` + `exceptions.md` + relevant domain file |
| **Auth Work** | "auth", "login", "jwt", "rbac", "permissions", "token", "password" | `security.md` + `api-design.md` |
| **Database Work** | "model", "migration", "query", "repository", "table", "schema" | `database.md` + `architecture.md` |
| **Bug Fix / Debug** | "fix", "error", "broken", "500", "422", "not returning", "crash" | `common-errors.md` + relevant domain file |
| **Testing** | "test", "coverage", "fixture", "pytest", "mock" | `testing.md` + relevant domain file |
| **Background Jobs** | "task", "celery", "queue", "scheduled", "worker", "cron" | `background-tasks.md` |
| **Logging** | "log", "trace", "monitoring", "debug output", "request tracking" | `logging.md` |
| **Refactor** | "refactor", "clean up", "restructure", "reorganize", "simplify" | `architecture.md` + relevant files |

### Resolving "relevant domain file"

Scan the user's request for domain keywords and map them to specific reference files:

- Database/models/queries/migrations/repository keywords → `database.md`
- Auth/tokens/permissions/login/security keywords → `security.md`
- Response format/schemas/endpoints keywords → `api-design.md`
- Error handling/exceptions/status codes keywords → `exceptions.md`
- Logging/tracing/monitoring keywords → `logging.md`
- Background jobs/tasks/workers keywords → `background-tasks.md`
- Testing/fixtures/coverage keywords → `testing.md`

If multiple domains apply, load up to **3 total reference files maximum**, prioritizing the most specific domain to the task at hand.

---

## Reference Loading Instructions

Use the Read tool to load ONLY the reference files identified in the classification above. Files are located at `${CLAUDE_PLUGIN_ROOT}/references/`. Read each file completely. Never read all 9 files at once — load only what the task requires (1-3 files).

Available reference files:
- `${CLAUDE_PLUGIN_ROOT}/references/architecture.md`
- `${CLAUDE_PLUGIN_ROOT}/references/api-design.md`
- `${CLAUDE_PLUGIN_ROOT}/references/database.md`
- `${CLAUDE_PLUGIN_ROOT}/references/security.md`
- `${CLAUDE_PLUGIN_ROOT}/references/exceptions.md`
- `${CLAUDE_PLUGIN_ROOT}/references/logging.md`
- `${CLAUDE_PLUGIN_ROOT}/references/testing.md`
- `${CLAUDE_PLUGIN_ROOT}/references/background-tasks.md`
- `${CLAUDE_PLUGIN_ROOT}/references/common-errors.md`

---

## Phase Process

Every task goes through phases in order. No skipping. No combining.

Flow: Phase 1 → Phase 2 → Phase 3 → Phase 4 → Phase 5 → Phase 6
Vague requests: Phase 0 → Phase 1 → Phase 2 → Phase 3 → Phase 4 → Phase 5 → Phase 6

---

### Phase 0: Clarify (vague requests only)

This phase runs ONLY when the request is classified as Vague/Unclear. It runs before the main pipeline. Do not load any reference files yet — you need to understand the problem first.

1. Read the existing codebase — use Glob to find the project structure, check for recent errors in logs or test output, review recent git changes with `git log --oneline -10` and `git diff`.
2. Determine what the actual problem or need is based on what you find.
3. Re-classify the task into one of the specific types in the classification table above.
4. Proceed to Phase 1 with the clarified classification.

Do not ask the user to clarify if you can determine the need by reading the codebase. Only ask the user directly if the codebase provides insufficient context to determine the task type.

---

### Phase 1: Classify

1. Parse the user's request — extract key terms, identify the domain, and determine what outcome the user wants.
2. Match against the task classification table above. Use the trigger signals column to find the best fit.
3. If the request is vague or unclear (matches no specific type, or matches multiple types ambiguously), route to Phase 0 first.
4. Identify which reference files to load (1-3 files max) using the "References to Load" column and the domain file heuristic.
5. Proceed to Phase 2.

---

### Phase 2: Load References

1. Use the Read tool to load each reference file identified in Phase 1.
2. Files are at `${CLAUDE_PLUGIN_ROOT}/references/`.
3. Read each file completely before proceeding.
4. Do not proceed to Phase 3 until all identified reference files have been read.

---

### Phase 3: Analyze

**HARD GATE: Do NOT write any code until this phase is complete. Do NOT skip reading the reference files. If a reference file has a pattern for your task, USE IT. Do not invent your own version.**

1. Read the existing codebase — the relevant files, not everything. Use Glob to find project structure (`**/*.py`, `**/requirements*.txt`, `**/pyproject.toml`) and Grep to find specific patterns (existing routes, models, services).
2. Understand the current architecture, patterns, and conventions already in use. Identify which layers exist (endpoints, services, repositories, models) and how they interact.
3. For bug fixes: identify the ROOT CAUSE, not the symptom. Trace the error to its origin. Read stack traces carefully. Check if the error is caused by a pattern mismatch, a missing dependency, or a logic error.
4. For new features: understand how the feature fits into the existing system — which layer it touches, what dependencies it needs, what existing patterns it should follow. Identify all files that need to change.
5. Compare what the reference files specify with what the codebase currently does. Note any gaps or deviations. The reference patterns take precedence for new code; existing code should be refactored toward reference patterns only when the user explicitly asks for refactoring.

---

### Phase 4: Design

1. Propose an approach based on the reference patterns combined with the existing codebase analysis.
2. Explain WHY this approach — not just what you will do, but why it is the right choice given the current system state.
3. Present the design to the user for approval before proceeding.
4. If the approach requires changing existing patterns, explain the trade-offs explicitly.

**If the user rejects the design:** Return to Phase 3 with the user's feedback as additional context. Re-analyze the problem with this new information. Propose a revised approach. Repeat until the user approves. Do NOT proceed to Phase 5 without explicit approval.

---

### Phase 5: Implement

Follow the reference patterns exactly. Match existing codebase conventions. Create files in the correct layers — endpoints in the router/endpoint layer, business logic in services, data access in repositories. Never put business logic in endpoint functions. Never import HTTP concepts (Request, Response, status codes) in service or repository layers.

#### The Root-Cause Protocol

When encountering ANY problem during implementation:

> 1. **STOP.** Do not write a single line of fix code.
> 2. **WHY does this problem exist?** Trace to origin. Is it a symptom of something deeper?
> 3. **IS a fix even the right move?** Maybe the approach needs rethinking. Maybe the "problem" is correct behavior and the expectation is wrong.
> 4. **What is the HOLISTIC solution?** Not "what makes this error go away" but "what makes the system correct."
> 5. **Present analysis to the user.** Explain what you found, why it exists, what the root solution is.
> 6. **Only then implement.** With understanding, not with a patch.

**THERE ARE NO QUICK FIXES. THERE ARE NO AUTO-FIXES. EVERY PROBLEM IS AN INVESTIGATION. EVERY FIX IS A ROOT-CAUSE FIX. IF YOU CATCH YOURSELF WRITING A PATCH, STOP AND INVESTIGATE.**

---

### Phase 6: Validate

Run 3-level verification on everything you implemented. Do not skip this phase. Do not declare the task complete until all three levels pass.

1. **Exists** — Did you create all the files and functions needed? Use Glob and Grep to verify. Check that every file mentioned in the design was actually created and every function signature is present.
2. **Substantive** — Are they real implementations? Not stubs, not TODOs, not placeholders, not "implement this later" comments. Read each created file and confirm it contains real, working code with actual logic.
3. **Wired** — Are they actually connected to the system? Route registered in the router and included in the app? Dependency injected via Depends()? Migration file created and ready to run? Tests import the correct modules and can be discovered by pytest? Configuration values added to settings?

#### Validation Checklist

- [ ] Response format matches the standard (`success`/`code`/`message`/`data`/`errors`)
- [ ] Error handling follows exceptions.md patterns
- [ ] No blocking calls in async code
- [ ] Layer compliance (no business logic in endpoints, no HTTP concepts in services)
- [ ] Tests cover the new code
- [ ] No hardcoded secrets or credentials

---

## Rationalization Prevention

| If you're thinking... | The truth is... |
|----------------------|-----------------|
| "I know FastAPI well enough, I don't need the reference file" | The reference file has THIS PROJECT'S patterns. Read it. |
| "This is a simple change, skip the analysis" | Simple changes in the wrong pattern cause cascading fixes. Analyze. |
| "I'll add the tests later" | Phase 6 verification will catch you. Write them now. |
| "This pattern is close enough" | Close enough = inconsistent codebase. Match the reference exactly. |
| "Let me just quickly patch this" | Quick patches are explicitly forbidden. Find the root cause. |
| "I can auto-fix this small bug" | No. Investigate why it exists. Then fix holistically. |
