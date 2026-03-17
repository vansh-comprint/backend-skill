<p align="center">
  <img src="https://capsule-render.vercel.app/api?type=waving&color=0:0d1117,50:161b22,100:1f6feb&height=220&section=header&text=Comprint%20Skills&fontSize=52&fontColor=ffffff&animation=fadeIn&fontAlignY=35&desc=Claude%20Code%20Plugin%20Marketplace&descSize=18&descAlignY=55&descColor=8b949e" width="100%"/>
</p>

<p align="center">
  <a href="#quick-start"><img src="https://img.shields.io/badge/Get_Started-blue?style=for-the-badge" alt="Get Started"></a>
  <a href="#plugins"><img src="https://img.shields.io/badge/Browse_Plugins-purple?style=for-the-badge" alt="Browse Plugins"></a>
  <a href="#contributing"><img src="https://img.shields.io/badge/Contribute-orange?style=for-the-badge" alt="Contribute"></a>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/plugins-2-blue?style=flat-square&logo=data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHdpZHRoPSIyNCIgaGVpZ2h0PSIyNCIgdmlld0JveD0iMCAwIDI0IDI0IiBmaWxsPSJub25lIiBzdHJva2U9IndoaXRlIiBzdHJva2Utd2lkdGg9IjIiPjxyZWN0IHg9IjMiIHk9IjMiIHdpZHRoPSI3IiBoZWlnaHQ9IjciIHJ4PSIxIi8+PHJlY3QgeD0iMTQiIHk9IjMiIHdpZHRoPSI3IiBoZWlnaHQ9IjciIHJ4PSIxIi8+PHJlY3QgeD0iMyIgeT0iMTQiIHdpZHRoPSI3IiBoZWlnaHQ9IjciIHJ4PSIxIi8+PHJlY3QgeD0iMTQiIHk9IjE0IiB3aWR0aD0iNyIgaGVpZ2h0PSI3IiByeD0iMSIvPjwvc3ZnPg==">
  <img src="https://img.shields.io/badge/python-3.12+-3776AB?style=flat-square&logo=python&logoColor=white">
  <img src="https://img.shields.io/badge/license-MIT-green?style=flat-square">
  <img src="https://img.shields.io/badge/claude_code-compatible-8A2BE2?style=flat-square">
</p>

---

## Quick Start

```bash
/plugin marketplace add vansh-comprint/comprint-skills

# Install what you need
/plugin install comprint-back
/plugin install comprint-marketing
```

---

## Plugins

### ⚡ comprint-back

**The complete FastAPI development system.**

One command handles your entire backend — from scaffolding a new project to debugging a 422 error. It carries 9 curated knowledge areas and loads only what's relevant to your task.

```
/comprint-back scaffold a new FastAPI project with auth and database
/comprint-back add a CRUD endpoint for products with pagination
/comprint-back set up JWT with role-based access control
/comprint-back fix this 422 validation error
```

<details>
<summary><b>🔍 See all 9 knowledge areas</b></summary>

<br>

| | Domain | What's inside |
|---|--------|--------------|
| 🏗️ | **Architecture** | 4-layer pattern, simple + modular project layouts, dependency injection |
| 🔌 | **API Design** | Standardized 5-field responses, CRUD patterns, Pydantic schemas, versioning |
| 🗃️ | **Database** | Async SQLAlchemy 2.0, generic repositories, Alembic migrations, query optimization |
| 🔐 | **Security** | JWT access + refresh tokens, RBAC, CORS, rate limiting, security audit checklist |
| ⚠️ | **Errors** | Exception hierarchy, global handlers, validation formatting, FE-friendly responses |
| 📋 | **Logging** | Structured JSON, request ID tracing, ContextVar propagation |
| 🧪 | **Testing** | Async fixtures, FakeRepository (no mocking), integration tests, coverage config |
| ⏱️ | **Background Tasks** | Celery vs BackgroundTasks, scheduling, Docker Compose workers |
| 🐛 | **Common Errors** | 10 prevention patterns — Pydantic v2 gotchas, async traps, CORS, form quirks |

</details>

<details>
<summary><b>🧠 How smart routing works</b></summary>

<br>

The plugin doesn't dump everything into context. It classifies your request and loads only the relevant knowledge.

| You say | It loads |
|---------|----------|
| "add authentication" | `security.md` + `api-design.md` |
| "new project" | `architecture.md` + `database.md` + `logging.md` |
| "fix this 500" | `common-errors.md` + `exceptions.md` |
| "something's broken" | Investigates first, then decides |

</details>

<details>
<summary><b>🛠️ Built on</b></summary>

<br>

FastAPI · Pydantic v2 · SQLAlchemy 2.0 async · PostgreSQL · AsyncPG · Alembic · pytest · httpx · Celery · Redis · JWT · bcrypt

</details>

<br>

> 📖 [Full documentation →](plugins/comprint-back/README.md)

---

### 📣 comprint-marketing

**Marketing Agency OS — run your entire social media operation from Claude Code.**

16 commands covering brand onboarding, content generation, competitive intelligence, growth tracking, and cross-platform repurposing. Every command reads your brand's 10 knowledge files for fully contextual output.

```
/brand-onboard my-brand
/brand-generate my-brand
/competitor-intel my-brand
/weekly-review my-brand
```

<details>
<summary><b>📋 All 16 commands</b></summary>

<br>

| Category | Command | What it does |
|----------|---------|-------------|
| **Brand** | `/brand-onboard` | Deep onboarding — builds 10 knowledge files for a new brand |
| | `/growth-status` | Quick dashboard of brand health and level |
| | `/weekly-review` | Level-adapted weekly growth brief |
| | `/client-report` | Client-facing weekly report |
| | `/brand-audit` | 48-hour prospect audit for new clients |
| **Content** | `/brand-generate` | Full content engine — reels, carousels, stories, posts |
| | `/scripter` | Hinglish reel scriptwriter with hooks and CTAs |
| | `/repurpose` | Cross-platform content adaptation |
| | `/trend-scan` | Trending topics + content ideas |
| **Intel** | `/competitor-intel` | Intelligence brief with Steal Cards |
| | `/ig-research` | Deep Instagram account research |
| | `/ig-ai` | Multi-agent Instagram analysis |
| | `/yt-research` | YouTube channel research |
| | `/yt-ai` | YouTube content analysis |
| | `/yt-knowledge` | YouTube knowledge extraction |
| | `/yt-prompts` | YouTube prompt library |

</details>

<details>
<summary><b>🧠 Brand Knowledge System</b></summary>

<br>

Every brand gets 10 structured files that all commands read for context:

Brand · Voice · Design · Content · Audience · Platform · Performance · Growth · Competitors · Offer

Plus a shared **Content Intelligence Framework** covering hook engineering, story architecture, viral mechanics, and platform execution rules.

</details>

<br>

> 📖 [Full documentation →](plugins/comprint-marketing/README.md) · 🔗 [Full ecosystem with scripts & automation →](https://github.com/marketing-alisha/Marketing)

---

## Contributing

Want to add a plugin? Drop it in `plugins/your-plugin/` and open a PR.

```
plugins/
├── comprint-back/          ← FastAPI backend
├── comprint-marketing/     ← Marketing agency OS
├── your-plugin/            ← yours
│   ├── .claude-plugin/
│   │   └── plugin.json
│   ├── commands/
│   ├── skills/
│   └── references/
```

---

<p align="center">
  <img src="https://capsule-render.vercel.app/api?type=waving&color=0:0d1117,50:161b22,100:1f6feb&height=120&section=footer" width="100%"/>
</p>

<p align="center">
  <sub>Built by <a href="https://github.com/vansh-comprint">Comprint</a></sub>
</p>
