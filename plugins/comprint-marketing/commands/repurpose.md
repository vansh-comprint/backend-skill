---
description: "Take one content idea and adapt it for multiple platforms (Instagram, LinkedIn, YouTube). Write once, publish everywhere."
---

# /repurpose — Cross-Platform Content Adapter

> One idea, multiple platforms. Adapts a single content concept into platform-optimized versions.

## Usage

```
/repurpose <brand-name> "<content idea or topic>"
/repurpose <brand-name> --from-post <url>           (adapt an existing post)
/repurpose <brand-name> --from-trend <trend-name>   (from latest trend scan)
```

## How It Works

### Step 1: Load Brand Context

Read these files:
- `D:/Marketing/brands/$BRAND/BRAND.md` — identity, handle
- `D:/Marketing/brands/$BRAND/VOICE.md` — tone calibration per platform
- `D:/Marketing/brands/$BRAND/DESIGN.md` — visual system
- `D:/Marketing/brands/$BRAND/CONTENT.md` — pillars, hooks
- `D:/Marketing/brands/$BRAND/AUDIENCE.md` — ICP (may differ by platform)
- `D:/Marketing/brands/$BRAND/PLATFORM.md` — platform-specific rules
- `D:/Marketing/brands/$BRAND/OFFER.md` — CTA language

### Step 1.5: Content Intelligence Layer

Read `D:/Marketing/brands/_template/CONTENT-INTELLIGENCE.md` before generating platform versions.

Apply when adapting content across platforms:

1. **Platform Execution Rules:** Each platform version must follow the platform-specific rules from Category 6 of CONTENT-INTELLIGENCE.md. Do NOT copy-paste across platforms — each gets native formatting, native tone, and native structure.

2. **Awareness Calibration per Platform:** Different platforms may have different audience awareness levels. LinkedIn audience is typically more Solution/Product Aware. Instagram is typically Problem/Solution Aware. YouTube varies by content type. Adjust message framing accordingly using Category 3 rules.

3. **Hook Re-engineering per Platform:**
   - Instagram: Visual-first hook (what they SEE in first frame/first line)
   - LinkedIn: Text-first hook (must work in 2 lines before "see more" truncation)
   - YouTube: Thumbnail + title hook (CTR-optimized, face + text + context)
   Each platform gets a DIFFERENT hook even for the same topic. Use Hook Engineering formulas from Category 1, selecting based on platform conventions.

4. **Viral Mechanic per Platform:**
   - Instagram: Optimize for shares (relatability) OR saves (reference value)
   - LinkedIn: Optimize for comments (dwell time + engagement question CTA)
   - YouTube: Optimize for watch time (AVD — open loops, pattern stacking)
   Apply matching triggers from Category 5.

5. **Conversion Triggers per Platform:** Read brand level from GROWTH.md and offer status from OFFER.md. Apply level-gated CTAs from Category 4. LinkedIn CTAs should be softer than Instagram. YouTube CTAs should be embedded throughout, not just at end.

### Step 1.6: Adaptation Preview (confirm before finalizing)

After planning the platform adaptations but BEFORE writing final versions, show the operator a quick summary:

```
Here's how I'm adapting this for each platform:
- **Instagram:** [hook style] → optimizing for [shares/saves]
- **LinkedIn:** [hook style] → optimizing for [comments]
- **YouTube:** [hook style] → optimizing for [watch-time]

Confirm, or tell me what to adjust.
```

Wait for confirmation. If the operator adjusts (e.g., "make LinkedIn more casual" or "optimize IG for saves instead"), apply the change before generating.

---

### Step 2: Understand the Source Content

If given a topic/idea: Define the core message, key value, and target audience segment.
If given a URL: Read/analyze the existing post to extract the core message.
If given a trend: Read the latest `YYYY-WXX-trends.md` from vault.

Define:
- **Core message:** The ONE thing the audience should take away
- **Value type:** Educational / Entertaining / Inspirational / Promotional
- **Best pillar:** Which content pillar from CONTENT.md this serves

### Step 3: Generate Platform Versions

For each platform, adapt the content while keeping the core message identical. The FORMAT and TONE change, not the value.

#### Instagram Version

Choose the best format for this content:
- **Reel** (if the content is a story, demo, or hot take) — use /scripter hook frameworks
- **Carousel** (if the content is educational, list-based, or step-by-step)
- **Static** (if the content is a bold quote or single insight)
- **Story** (if the content is time-sensitive or behind-the-scenes)

Generate the full content piece with:
- Hook (Instagram-optimized: visual + first 3 seconds or first line)
- Full script/copy
- Visual direction from DESIGN.md
- CTA from OFFER.md
- 5 hashtags (Instagram max)
- Best posting time from PLATFORM.md

#### LinkedIn Version

Choose the best format:
- **Text post** (if the content is a hot take, lesson, or story)
- **Document carousel** (if the content is educational or list-based)
- **Article** (if the content is deep/thought-leadership)

Generate with:
- Hook (LinkedIn-optimized: first 2 lines before "see more")
- Body with generous whitespace (short paragraphs, line breaks)
- Professional but human tone (shift VOICE.md toward professional register)
- Engagement question at the end
- Soft CTA
- 3-5 hashtags at end

#### YouTube Version

Choose the best format:
- **Short** (if under 60 seconds — similar to Reel)
- **Video script** (if 3-10 minute educational content)
- **Community post** (if text/poll format)

Generate with:
- Title (keyword-rich, curiosity-driven)
- Thumbnail concept (text + visual from DESIGN.md)
- Script with timestamps
- Description with hooks, links, tags
- 5-10 YouTube tags

### Step 4: Platform Comparison Table

```markdown
## Repurpose Summary: "[Topic]"

| Element | Instagram | LinkedIn | YouTube |
|---------|-----------|----------|---------|
| Format | [Reel/Carousel/...] | [Post/Carousel/...] | [Short/Video/...] |
| Hook | "[first line]" | "[first line]" | "[title]" |
| Length | [X seconds / X slides] | [X characters] | [X minutes] |
| CTA | [IG CTA] | [LI CTA] | [YT CTA] |
| Hashtags | 5 | 3-5 | 5-10 tags |
| Best time | [from PLATFORM.md] | Tue-Thu 8-10am | [from PLATFORM.md] |
| Priority | [1/2/3] | [1/2/3] | [1/2/3] |
```

### Step 5: Posting Schedule

Suggest a staggered posting schedule:
- **Day 1:** Post on the PRIMARY platform (where the content fits best)
- **Day 2:** Post adapted version on SECONDARY platform
- **Day 3:** Post on TERTIARY platform
- Never post the same content on all platforms the same day.

### Step 6: Offer Next Steps

- "Want me to generate more content on this topic?" → `/brand-generate`
- "Want me to scan for more trending topics?" → `/trend-scan`
- "Want me to write the full reel script?" → `/scripter`
