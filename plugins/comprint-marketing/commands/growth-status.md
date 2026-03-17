---
description: "Quick anytime growth status check. Read-only dashboard showing current level, fundamentals score, unlock progress, and offer status. No scripts, no file modifications — just instant visibility. Run: /growth-status <brand-name>"
---

# Growth Status: Quick Dashboard

**Brand to check:** $ARGUMENTS

---

## COGNITIVE OVERRIDE: Dashboard Mode

> You are a **status display**. Fast, clean, accurate. Read three files, compute the display, print the dashboard. No analysis, no recommendations, no essays. The operator wants a glance — give them a glance.

---

## EXECUTION PROTOCOL

### Step 1: Read Brand Files

Read these three files (read-only — do NOT modify any files):

1. `brands/$ARGUMENTS/GROWTH.md` — current level, fundamentals score, unlock progress, streak
2. `brands/$ARGUMENTS/OFFER.md` — offer status (DRAFT / DEFINED / TESTED / PROVEN)
3. `brands/$ARGUMENTS/PERFORMANCE.md` — followers, engagement rate, latest metrics

If `brands/$ARGUMENTS/` does not exist, tell the operator:
> "Brand '$ARGUMENTS' not found. Run `/brand-onboard $ARGUMENTS` to set it up."

If any individual file is missing, use "N/A" for its fields and note which file is missing.

### Step 2: Extract Data

From **GROWTH.md**, extract:
- `current_level` — L1, L2, or L3
- `level_name` — FOUNDATION, OPTIMIZATION, or INTELLIGENCE
- `followers` — current follower count
- `total_posts` — total posts published
- `fundamentals_score` — X/5
- `fundamentals_grade` — GREEN, YELLOW, or RED
- `fundamentals_week` — date of last fundamentals check
- Unlock criteria progress (L1->L2: posts, bio link, offer; L2->L3: weeks analysis, KEEP recs)
- Historical streak — last 4 weeks of scores

From **OFFER.md**, extract:
- `offer_status` — DRAFT, DEFINED, TESTED, or PROVEN

From **PERFORMANCE.md**, extract:
- `followers` — use this as primary if available (scraped data is more accurate)
- `engagement_rate` — current engagement rate percentage
- `avg_likes` — average likes per post
- `avg_comments` — average comments per post
- Growth tracking entries — for week-over-week delta

### Step 3: Compute Display Values

1. **Followers:** Prefer PERFORMANCE.md value over GROWTH.md value.
2. **Fundamentals color:** GREEN (5/5), YELLOW (3-4/5), RED (0-2/5).
3. **Unlock progress percentage:**
   - For L1->L2: Calculate based on the 3 criteria (posts published / 90, bio link, offer defined). Weight posts at 60%, bio link at 20%, offer at 20%. Compute overall percentage.
   - For L2->L3: Calculate based on the 2 criteria (weeks analysis / 4, KEEP recs / 3). Equal weight.
4. **Progress bar:** Build a 10-character bar using filled and empty blocks based on percentage.
5. **Level-up proximity:** If overall unlock progress is >75%, flag as "CLOSE TO LEVEL UP".
6. **Last review date:** Check for the most recent file in `vault/01-Research/weekly-briefs/$ARGUMENTS/` matching `*-brief.md` pattern. If found, extract the date. If none, show "No reviews yet".

### Step 4: Print Dashboard

Print this exact dashboard format (replace all bracketed values with real data):

```
=============================================
  [BRAND] -- Growth Status
=============================================
  Level:        L[X] [LEVEL_NAME]
  Followers:    [N]
  Engagement:   [X.XX]%
  Fundamentals: [X]/5 ([GRADE])
  Offer Status: [STATUS]
---------------------------------------------
  Unlock Progress (L[X] -> L[X+1]):
    [Criterion 1]:  [current]/[target]  [progress_bar] [pct]%
    [Criterion 2]:  [status_icon]
    [Criterion 3]:  [status_icon]
---------------------------------------------
  Last 4 Weeks:
    [W1]: [score]/5 [GRADE]
    [W2]: [score]/5 [GRADE]
    [W3]: [score]/5 [GRADE]
    [W4]: [score]/5 [GRADE]
---------------------------------------------
  Last Weekly Review: [date or "None yet"]
  Focus: [Level mantra]
=============================================
```

#### Status Icons
- Use `[PASS]` for met criteria
- Use `[FAIL]` for unmet criteria
- Use `[N/A]` for unavailable data

#### Progress Bar Format
Build a visual progress bar like this:
- 0%:   `[__________]`
- 28%:  `[===_______]`
- 50%:  `[=====_____]`
- 75%:  `[========__]`
- 100%: `[==========]`

Each `=` represents 10%. Round to nearest 10%.

### Step 5: Level-Up Alert

If unlock progress is above 75%, append this after the dashboard:

```
  >> LEVEL UP APPROACHING: [X]% of L[N+1] criteria met.
  >> Remaining: [list what's still needed]
```

### Step 6: Red Alert

If fundamentals grade is RED, append this:

```
  !! FUNDAMENTALS RED: Score [X]/5.
  !! Top priority: [name the #1 failed fundamental]
```

---

## IMPORTANT RULES

1. **READ-ONLY.** This skill does NOT modify any files. No writes, no script execution, no updates.
2. **Instant.** Just read 3 files and display. No web searches, no scraping, no agents.
3. **Concise.** Print the dashboard and nothing else. No analysis paragraphs, no recommendations. If the operator wants depth, they should run `/weekly-review`.
4. **Accurate.** If a value can't be determined, show "N/A" rather than guessing.
5. **Graceful degradation.** If one file is missing, still show what you can from the others. Note the missing file at the bottom.
