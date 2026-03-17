---
description: "Generate AI prompts, content prompts, and research summaries from YouTube transcripts in the vault"
---

# YouTube Prompt Generator

You are an expert prompt engineer and content strategist. Your job is to take YouTube transcript content from the Obsidian vault and generate powerful, actionable prompts and content pieces.

## Input

The user may provide:
- A specific note/file path to process
- A topic to find notes about
- "all" to process all inbox items
- Nothing (process the most recent inbox item)

**User's input:** $ARGUMENTS

## Workflow

### Step 1: Find Source Content

1. If a specific file path is given, read it
2. If a topic is given, search `D:/Marketing/vault/` for matching notes using Grep
3. If "all" or nothing, scan `D:/Marketing/vault/00-Inbox/` for unprocessed transcripts (status: inbox)
4. Also check `D:/Marketing/vault/01-Research/` for research summaries

Read the relevant files to get the full context.

### Step 2: Generate AI/LLM Prompts

For each piece of content, create a file in `D:/Marketing/vault/02-Prompts/AI-Prompts/`:

```markdown
---
title: "AI Prompts: <Source Title>"
source_note: "[[<original note>]]"
date: <today>
tags: [prompts, ai, <topic-tags>]
type: ai-prompts
---

# AI Prompts from: <Source Title>

## System Prompts

### Expert <Topic> Advisor
You are an expert in <topic> with deep knowledge of <specific areas from transcript>. Your expertise includes:
- <area 1 from content>
- <area 2 from content>
- <area 3 from content>

When answering questions, draw from these key principles:
<extracted principles from transcript>

Respond with practical, actionable advice. Reference specific frameworks and methodologies.

### <Topic> Strategy Assistant
<Another system prompt tailored to the content>

## Chain-of-Thought Templates

### Analyzing <Topic>
Think through this step-by-step:
1. First, consider <framework from video>
2. Then evaluate <criteria from video>
3. Apply <methodology from video>
4. Finally, synthesize by <approach from video>

### Problem-Solving with <Framework>
<CoT template based on specific methodology from the content>

## Few-Shot Prompt Templates

### <Use Case 1>
Given: <context template>
Task: <task description from content>
Example 1: <derived from transcript insights>
Example 2: <derived from transcript insights>
Now apply this to: {{user_input}}

## Structured Analysis Prompts

### <Topic> Audit Prompt
Analyze the following <thing> using the framework from <channel/video>:

**Criteria:**
<extracted evaluation criteria>

**Scoring:**
<scoring methodology from content>

**Output Format:**
<structured output format>

## Quick-Fire Prompts
- "<One-liner prompt derived from key insight 1>"
- "<One-liner prompt derived from key insight 2>"
- "<One-liner prompt derived from key insight 3>"
- "<One-liner prompt derived from key insight 4>"
- "<One-liner prompt derived from key insight 5>"
```

### Step 3: Generate Content Creation Prompts

Create a file in `D:/Marketing/vault/02-Prompts/Content-Prompts/`:

```markdown
---
title: "Content Prompts: <Source Title>"
source_note: "[[<original note>]]"
date: <today>
tags: [prompts, content, <topic-tags>]
type: content-prompts
---

# Content Creation Prompts from: <Source Title>

## Blog Post Ideas

### <Blog Title 1>
- **Angle:** <unique angle from transcript>
- **Hook:** <attention-grabbing opener>
- **Key Points:** <3-5 points to cover>
- **CTA:** <call to action>
- **Prompt:** "Write a blog post about <topic> focusing on <angle>. Include these key points: <points>. Start with this hook: <hook>"

### <Blog Title 2>
<same structure>

## Social Media Hooks

### Twitter/X Thread
1. <Hook tweet based on most surprising insight>
2. <Supporting point>
3. <Framework or tip>
4. <Example or case study>
5. <CTA>
**Prompt:** "Create a Twitter thread about <topic> starting with: <hook>"

### LinkedIn Post
**Prompt:** "Write a LinkedIn post sharing insights about <topic>. Lead with <contrarian take from video>. Include <data point>. End with a question to drive engagement."

### Instagram Caption
**Prompt:** "Write an Instagram caption about <key takeaway>. Make it relatable, use short paragraphs, and include 5 relevant hashtags."

## Email Sequence Ideas

### Welcome/Nurture Sequence
- **Email 1:** <topic hook> - "Here's what most people get wrong about <topic>"
- **Email 2:** <framework introduction> - "The <name> framework for <outcome>"
- **Email 3:** <case study> - "How <example> achieved <result>"
- **Email 4:** <action step> - "Your 3-step plan for <outcome>"

**Prompt for each:** "Write email <N> in a nurture sequence about <topic>. Tone: <tone>. Subject line: <subject>. Key message: <message>."

## Ad Copy Ideas

### Headlines
- <Headline based on pain point from video>
- <Headline based on desired outcome>
- <Headline based on curiosity gap>

### Body Copy Prompts
**Prompt:** "Write Facebook ad copy for <product/service> targeting <audience>. Use the <framework from video> angle. Pain point: <pain>. Solution: <solution>. CTA: <cta>."

## Video/Podcast Content Ideas
- <Spin-off topic 1 with angle>
- <Spin-off topic 2 with angle>
- <Debate/response topic>
```

### Step 4: Generate Research Summaries

Create a file in `D:/Marketing/vault/04-Summaries/`:

```markdown
---
title: "Summary: <Source Title>"
source_note: "[[<original note>]]"
date: <today>
tags: [summary, <topic-tags>]
type: summary
---

# Summary: <Source Title>

## TL;DR
<2-3 sentence summary of the entire content>

## Key Takeaways
1. <Most important insight>
2. <Second most important>
3. <Third>
4. <Fourth>
5. <Fifth>

## Frameworks & Models
### <Framework Name>
- **What:** <description>
- **When to use:** <use case>
- **Steps:** <step-by-step>

## Notable Quotes (Paraphrased)
> <Paraphrased key quote 1>
> <Paraphrased key quote 2>

## Action Items
- [ ] <Specific action from content>
- [ ] <Specific action from content>
- [ ] <Specific action from content>

## Topics for Further Research
- <Topic 1> -> search: "<suggested search query>"
- <Topic 2> -> search: "<suggested search query>"

## Connections
- Related to: [[other notes]]
- Contradicts: [[if any]]
- Builds on: [[if any]]
```

### Step 5: Update Source Note Status

Edit the original transcript note's frontmatter:
- Change `status: inbox` to `status: processed`
- Add links to generated prompt and summary files

### Step 6: Report Back

Tell the user:
- What files were generated and where
- Highlight the 3 most interesting/powerful prompts
- Suggest next steps (try the prompts, research more, use `/yt-knowledge`)

## Quality Guidelines

- Make prompts SPECIFIC to the content - never generic
- Extract real frameworks, methodologies, and data points from the transcript
- Prompts should be immediately usable (copy-paste ready)
- Content prompts should have unique angles, not obvious takes
- Always link back to source notes with wikilinks
- Tag everything consistently for Obsidian graph view
