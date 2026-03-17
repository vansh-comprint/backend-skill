# comprint-marketing

Marketing Agency OS for Claude Code — AI-powered brand management, content generation, competitive intelligence, and growth tracking.

## What It Does

A complete operating system for managing brands' social media presence. Covers the full agency workflow from onboarding a new brand to generating publication-ready content to tracking weekly growth.

## Commands

### Brand Management
| Command | What it does |
|---------|-------------|
| `/brand-onboard` | Deep onboarding system — builds 10 knowledge files for a new brand |
| `/growth-status` | Quick dashboard of brand health and growth level |
| `/weekly-review` | Level-adapted weekly growth brief with metrics and action items |
| `/client-report` | Client-facing weekly report ready to send |
| `/brand-audit` | 48-hour prospect audit for potential new clients |

### Content Generation
| Command | What it does |
|---------|-------------|
| `/brand-generate` | Full content generation engine — reels, carousels, stories, posts, captions |
| `/scripter` | Hinglish reel scriptwriter with hooks and CTAs |
| `/repurpose` | Cross-platform content adaptation |
| `/trend-scan` | Trending topics + content ideas for your brand |

### Intelligence & Research
| Command | What it does |
|---------|-------------|
| `/competitor-intel` | Intelligence brief with Steal Cards — adapted competitor hooks |
| `/ig-research` | Deep Instagram account research |
| `/ig-ai` | Multi-agent Instagram analysis |
| `/yt-research` | YouTube channel research |
| `/yt-ai` | YouTube content analysis |
| `/yt-knowledge` | YouTube knowledge extraction |
| `/yt-prompts` | YouTube prompt library |

## Brand Knowledge System

Every brand gets 10 structured knowledge files that the commands read for context:

| File | Contains |
|------|----------|
| `BRAND.md` | Positioning, personality, mission |
| `VOICE.md` | Tone, vocabulary, do's and don'ts |
| `DESIGN.md` | Colors, typography, visual rules |
| `CONTENT.md` | Pillars, content mix, hooks, hashtags |
| `AUDIENCE.md` | ICPs, pain points, awareness levels |
| `PLATFORM.md` | Algorithm rules, posting times, specs |
| `PERFORMANCE.md` | Metrics, benchmarks, tracking |
| `GROWTH.md` | Level (L1/L2/L3), fundamentals, streaks |
| `COMPETITORS.md` | Competitor roster and analysis log |
| `OFFER.md` | Value equation, CTAs, pricing |

Templates for all 10 files are included in this plugin at `references/brand-templates/`.

## Growth Level System

Brands progress through 3 levels — each unlocks new capabilities:

- **L1 Foundation** — Core content fundamentals, basic metrics
- **L2 Optimization** — Competitive intelligence, content optimization
- **L3 Intelligence** — Advanced analytics, viral mechanics, growth automation

## Getting Started

```bash
# Install the plugin
/plugin install comprint-marketing

# Onboard your first brand
/brand-onboard my-brand-name

# Generate content
/brand-generate my-brand-name
```

## Full System

This plugin provides the core commands and templates. For the full ecosystem with Python automation scripts, GitHub Actions workflows, and the Obsidian research vault, see the [Marketing OS repository](https://github.com/marketing-alisha/Marketing).

## Content Intelligence Framework

Included in `references/brand-templates/CONTENT-INTELLIGENCE.md` — covers hook engineering, story architecture, awareness calibration, conversion triggers, viral mechanics, and platform execution rules.
