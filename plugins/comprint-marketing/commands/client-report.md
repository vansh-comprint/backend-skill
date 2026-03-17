---
description: "Generate a professional weekly client report for any brand with performance metrics, insights, and next steps. Client-facing format — no internal jargon."
---

# Client Report Generator

**User's request:** /client-report `<brand-name>`

---

## EXECUTION PROTOCOL

### Phase 0: Parse Input

Extract `<brand-name>` from the user's request. Example: `/client-report ainxtstudio` → brand = "ainxtstudio"

### Phase 1: Gather Data Sources

Read the following files in order (stop if a file doesn't exist):

1. **Brand Profile:** `D:/Marketing/brands/<brand-name>/` (all .md files)
   - BRAND.md (brand identity, positioning, personality)
   - PERFORMANCE.md (metrics, benchmarks, historical data)
   - CONTENT.md (content pillars, mix targets)
   - GROWTH.md (current level, fundamentals score)
   - OFFER.md (offer status, CTA)
   - VOICE.md (tone — use this to match report voice)
   - AUDIENCE.md (ICP — know who the client serves)

2. **Weekly Intelligence (Most Recent):**
   - `D:/Marketing/vault/01-Research/weekly-briefs/<brand-name>/` (latest dated file)
   - OR if that doesn't exist, check `D:/Marketing/vault/00-Inbox/` for recent IG_ profile files for this brand

3. **Performance Tracking (Most Recent):**
   - Latest growth tracking entry in PERFORMANCE.md (the "Growth Tracking" table)

### Phase 2: Calculate Week-over-Week Changes

For each key metric in PERFORMANCE.md:
- Extract "This Week" value (most recent row in Growth Tracking table)
- Extract "Last Week" value (previous row)
- Calculate: Change = This Week - Last Week
- Calculate: Percent Change = (Change / Last Week) × 100, rounded to 1 decimal
- Format as "+N (+X.X%)" or "-N (-X.X%)"

If historical data is missing, note "N/A — first week of tracking"

### Phase 3: Synthesize Narrative Sections

Generate the following sections. Use the brand's actual data and recent activity.

#### Section 1: This Week at a Glance

Create a 5-row metrics table:

```
| Metric | This Week | Last Week | Change |
|--------|-----------|-----------|--------|
| Followers | [number] | [number] | [+/-N (+/-X.X%)] |
| Posts Published | [number] | [number] | [+/-N] |
| Avg Engagement Rate | [X.X%] | [X.X%] | [+/-X.X pp] |
| DMs Received | [number] | [number] | [+/-N] |
| Save Rate (Reels) | [X.X%] | [X.X%] | [+/-X.X pp] |
```

**Important:** If metric data doesn't exist for "last week", use "N/A" and note "establishing baseline"

#### Section 2: What We Did

Bullet-point narrative summarizing:
- **Content published:** Types (Reels, carousels, images), count, topics, any series continuations
- **Engagement activities:** Community responses, notable comments, DM interactions, Stories published
- **Strategy adjustments:** Any pivots or tests run this week (reference PERFORMANCE.md recommendations applied)
- Format: casual, client-friendly language. No internal jargon (no "L1/L2/L3", no "fundamentals score")

#### Section 3: What Worked

2-3 bullet points with:
- **Post example:** "Episode: [Name] — [Metric] engagement"
- **Why it worked:** Plain-English explanation (reference the "Why it worked" analysis from PERFORMANCE.md)
- **Pattern insight:** If multiple posts follow a pattern, extract it. Example: "Series format content gets 2.5x more repeat engagement"

#### Section 4: What We're Improving

2-3 bullet points with:
- **Honest assessment:** What didn't perform (reference "Worst Performer" or low-engagement posts)
- **Root cause:** Why (reference anti-patterns from PERFORMANCE.md — e.g., "Static images underperform vs Reels")
- **What's changing:** Specific adjustment for next week (e.g., "Moving educational content to carousel format")
- Tone: Constructive. "We're shifting strategy" not "This failed."

#### Section 5: Next Week's Plan

Bullet points covering:
- **Content calendar:** Number of posts planned, types (Reels/carousels/stories), general topics
- **Strategic focus:** What we're emphasizing (reference PERFORMANCE.md recommendations)
- **Tests running:** Any A/B tests or new formats being piloted (reference planned tests from PERFORMANCE.md)

#### Section 6: Growth Trajectory

Short narrative (3-4 sentences max):
- Current follower count and comparison to target milestone
- Engagement rate status and health signal
- Overall momentum ("gaining traction," "stabilizing," "ramping up")
- Plain language: "Building Phase," "Growth Phase," "Acceleration Phase" (instead of L1/L2/L3)

---

### Phase 4: Format & Polish

1. **Header:** Use template below
2. **Tables:** Markdown format, clean spacing
3. **Tone:** Professional but warm. Client-facing, no jargon.
4. **Length:** Aim for 1-1.5 pages (600-900 words max)
5. **No emojis** in section headers
6. **Data always cited:** Every metric references the brand's PERFORMANCE.md

### Phase 5: Output

Save as: `D:/Marketing/brands/<brand-name>/WEEKLY-REPORT-<YYYY-MM-DD>.md`

Return to user:
- File location
- Key insights summary (top 3 wins/lessons)
- Recommended client actions (pull from "Next Week's Plan")

---

## REPORT TEMPLATE

```markdown
# Weekly Report: [Brand Name]
> Week of [Start Date] – [End Date] | Prepared by [Agency Name]

---

## This Week at a Glance

| Metric | This Week | Last Week | Change |
|--------|-----------|-----------|--------|
| Followers | — | — | — |
| Posts Published | — | — | — |
| Avg Engagement Rate | — | — | — |
| DMs Received | — | — | — |
| Save Rate | — | — | — |

---

## What We Did

- [Bullet 1: Content published]
- [Bullet 2: Engagement activities]
- [Bullet 3: Strategy adjustments]

---

## What Worked

- [Top performer with why it worked]
- [Pattern insight]

---

## What We're Improving

- [Honest assessment + what's changing]
- [Adjustment for next week]

---

## Next Week's Plan

- [Content calendar overview]
- [Strategic focus]
- [Tests running]

---

## Growth Trajectory

[3-4 sentence narrative: current phase, momentum, next milestone]

---

**Questions?** Reply directly. We're here to hit your goals.
```

---

## TRANSLATION GUIDE (Internal Jargon → Client Language)

| Internal | Client Language |
|----------|-----------------|
| L1 FOUNDATION | Building Phase |
| L2 OPTIMIZATION | Growth Phase |
| L3 INTELLIGENCE | Acceleration Phase |
| Fundamentals Score | Consistency Score |
| Psychology Tags | (Don't mention) |
| Content Pillar Ratio | Content Mix |
| Engagement Hierarchy | Engagement Types (likes, comments, shares) |
| Algorithm Prioritization | Platform Performance |

