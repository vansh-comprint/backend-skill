---
description: "Scan trending topics, hooks, and content angles in your brand's niche. Returns actionable content ideas you can use THIS WEEK."
---

# /trend-scan — Trending Topic Scanner

> Finds what's trending in your niche right now and turns it into content ideas.

## Usage

```
/trend-scan <brand-name>
/trend-scan <brand-name> --platform instagram
/trend-scan <brand-name> --platform linkedin
/trend-scan <brand-name> --topic "AI automation"
```

## How It Works

### Step 1: Load Brand Context

Read these files to understand the brand's niche and audience:
- `D:/Marketing/brands/$ARGUMENTS/BRAND.md` — niche, positioning
- `D:/Marketing/brands/$ARGUMENTS/CONTENT.md` — content pillars
- `D:/Marketing/brands/$ARGUMENTS/AUDIENCE.md` — ICP, pain points
- `D:/Marketing/brands/$ARGUMENTS/COMPETITORS.md` — competitor handles

Extract: niche keywords, content pillars, ICP pain points, competitor names.

### Step 2: Research Trending Topics

Use web search to find what's trending in the brand's niche RIGHT NOW. Search for:

1. **Platform trends:**
   - Instagram: "trending [niche] reels 2026", "[niche] instagram trends this week"
   - LinkedIn: "trending [niche] linkedin posts", "viral [niche] linkedin"
   - YouTube: "trending [niche] videos", "[niche] youtube trends"

2. **Niche trends:**
   - "[niche] news this week"
   - "[niche] latest updates"
   - "what's new in [niche]"

3. **Competitor activity:**
   - Check competitor handles from COMPETITORS.md — what are they posting about?
   - Look for any viral posts in the last 7 days

4. **Seasonal/calendar hooks:**
   - What holidays, awareness days, or industry events are coming up?
   - Any product launches, algorithm changes, or platform updates?

### Step 3: Filter Through Brand Lens

For each trend found, evaluate:
- **Relevance:** Does it connect to at least one content pillar from CONTENT.md?
- **Audience fit:** Would the ICP from AUDIENCE.md care about this?
- **Brand angle:** Can this brand add a UNIQUE perspective? (check BRAND.md positioning)
- **Timeliness:** Is this still relevant or already fading?

Discard trends that don't pass at least 3 of 4 filters.

### Step 4: Generate Trend Report

Output this format:

```markdown
# Trend Scan — [brand-name]
**Date:** [YYYY-MM-DD]
**Niche:** [from BRAND.md]
**Platforms scanned:** [list]

---

## Hot Right Now (act this week)

### Trend 1: [Topic Name]
- **What's happening:** [1-2 sentence summary]
- **Why it matters to your audience:** [connect to ICP pain point]
- **Your angle:** [unique brand perspective]
- **Content ideas:**
  - Reel: "[hook idea]"
  - Carousel: "[title idea]"
  - LinkedIn post: "[hook idea]"
- **Pillar:** [which content pillar this serves]
- **Urgency:** HIGH / MEDIUM

### Trend 2: [Topic Name]
[same format]

### Trend 3: [Topic Name]
[same format]

---

## Emerging (watch this week, act next week)

### [Topic Name]
- **Signal:** [where you saw this trending]
- **When to act:** [timing recommendation]
- **Quick take:** [1 sentence brand angle]

---

## Evergreen Angles (always relevant, post anytime)

1. [Evergreen topic + hook idea]
2. [Evergreen topic + hook idea]
3. [Evergreen topic + hook idea]

---

## Hashtag Pulse

**Trending hashtags in your niche this week:**
| Hashtag | Platform | Volume Signal | Use? |
|---------|----------|--------------|------|
| #[tag] | IG/LI/YT | Rising/Stable/Fading | Yes/No |

**Recommended hashtag sets for this week:**
- Instagram (5 max): #tag1 #tag2 #tag3 #tag4 #tag5
- LinkedIn (3-5): #tag1 #tag2 #tag3
- YouTube tags: tag1, tag2, tag3, tag4, tag5
```

### Step 5: Save to Vault

Save the report to:
```
D:/Marketing/vault/01-Research/weekly-briefs/<brand-name>/YYYY-WXX-trends.md
```

### Step 6: Offer Next Steps

After presenting the report, offer:
- "Want me to generate content from any of these trends?" → chains to `/brand-generate`
- "Want me to deep-dive a competitor who's using this trend?" → chains to `post_deep_dive.py`
- "Want me to adapt these for a specific platform?" → chains to `/repurpose`
