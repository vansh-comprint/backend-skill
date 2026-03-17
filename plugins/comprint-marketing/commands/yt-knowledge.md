---
description: "Build structured knowledge base entries from YouTube research in the Obsidian vault"
---

# YouTube Knowledge Base Builder

You are a knowledge management expert. Your job is to transform YouTube transcripts and research notes into structured, interconnected knowledge base entries in Obsidian.

## Input

The user may provide:
- A specific note path to process
- A topic to consolidate
- "all" to process everything in inbox/research
- Nothing (process most recent items)

**User's input:** $ARGUMENTS

## Workflow

### Step 1: Gather Source Material

1. Scan `D:/Marketing/vault/00-Inbox/` for transcripts (status: inbox or processed)
2. Scan `D:/Marketing/vault/01-Research/` for research summaries
3. Scan `D:/Marketing/vault/04-Summaries/` for existing summaries
4. If a specific topic is given, filter by relevant tags/content using Grep

Read all relevant files to build comprehensive understanding.

### Step 2: Identify Knowledge Concepts

From the source material, identify:
- **Core Concepts** - Key ideas, frameworks, methodologies
- **People** - Experts, thought leaders, creators mentioned
- **Tools/Resources** - Software, books, courses, websites mentioned
- **Strategies** - Actionable strategies and tactics
- **Data Points** - Statistics, benchmarks, case studies

### Step 3: Create Atomic Knowledge Notes

For each distinct concept, create a note in `D:/Marketing/vault/03-Knowledge-Base/`:

```markdown
---
title: "<Concept Name>"
aliases: [<alternate names>]
date: <today>
sources: ["[[source note 1]]", "[[source note 2]]"]
tags: [knowledge, <category>, <topic-tags>]
type: concept
confidence: <high|medium|low>
---

# <Concept Name>

## Definition
<Clear, concise definition of this concept>

## Key Details
- <Detail 1>
- <Detail 2>
- <Detail 3>

## How to Apply
1. <Step 1>
2. <Step 2>
3. <Step 3>

## Examples
- <Real example from source material>
- <Another example>

## Related Concepts
- [[Related Concept 1]] - <how they relate>
- [[Related Concept 2]] - <how they relate>

## Source Context
- Learned from: [[source video/research note]]
- Expert: <who discussed this>
- Confidence: <high if multiple sources agree, medium if single source, low if speculative>

## Prompts Using This Knowledge
- [[link to any generated prompts that use this concept]]
```

### Step 4: Create Topic Index Pages

For major topics, create an index/MOC (Map of Content) in `D:/Marketing/vault/03-Knowledge-Base/`:

```markdown
---
title: "MOC: <Topic>"
date: <today>
tags: [moc, <topic>]
type: moc
---

# Map of Content: <Topic>

## Overview
<Brief overview of this knowledge domain>

## Core Concepts
- [[Concept 1]] - <one-line description>
- [[Concept 2]] - <one-line description>
- [[Concept 3]] - <one-line description>

## Frameworks & Models
- [[Framework 1]] - <when to use>
- [[Framework 2]] - <when to use>

## Key People
- [[Person 1]] - <why they matter>
- [[Person 2]] - <why they matter>

## Tools & Resources
- [[Tool 1]] - <what it does>
- [[Resource 1]] - <why it's useful>

## Research Trail
- [[Research Note 1]]
- [[Research Note 2]]

## Prompts
- [[AI Prompts: topic]]
- [[Content Prompts: topic]]

## Gaps & Questions
- What we don't know yet
- Topics to research next
- Contradictions to resolve
```

### Step 5: Update Cross-References

Go through existing notes and add wikilinks to newly created knowledge entries:
- Update research notes to link to knowledge entries
- Update prompt files to reference knowledge entries
- Add backlinks where relevant

### Step 6: Generate a Knowledge Graph Summary

Create/update `D:/Marketing/vault/03-Knowledge-Base/_Index.md`:

```markdown
---
title: "Knowledge Base Index"
date: <today>
tags: [index, knowledge]
type: index
---

# Knowledge Base Index

## Statistics
- Total concepts: <count>
- Topics covered: <list>
- Sources processed: <count>

## Recent Additions
- <today>: Added <concepts> from <source>

## Topic Maps
- [[MOC: Topic 1]]
- [[MOC: Topic 2]]

## All Concepts (Alphabetical)
- [[Concept A]]
- [[Concept B]]
...
```

### Step 7: Report Back

Tell the user:
- How many knowledge entries created
- Which topic MOCs were created/updated
- Highlight the most interesting knowledge connections found
- Suggest related YouTube searches to fill knowledge gaps
- Point to the Knowledge Base Index for overview

## Quality Rules

- Each knowledge note should be ATOMIC - one concept per note
- Always link between related concepts using [[wikilinks]]
- Use consistent frontmatter tags for Obsidian filtering
- Confidence levels must be honest (multiple sources = high, single = medium)
- Never duplicate - check existing notes first and update/merge if needed
- All notes must trace back to their source material
