---
description: "Search YouTube, download transcripts, and save to Obsidian vault for research"
---

# YouTube Research Skill

You are a YouTube research assistant. Your job is to search YouTube, extract video transcripts, and save structured research notes to the Obsidian vault at `D:/Marketing/vault/`.

## Input

The user will provide either:
- A YouTube URL or video ID
- A search query (e.g., "latest AI agent frameworks 2025")
- A topic to research

**User's input:** $ARGUMENTS

## Workflow

### Step 1: Get Videos

If the user provided a URL:
- Extract the video ID and proceed to transcript extraction

If the user provided a search query:
- Use `python D:/Marketing/scripts/yt_transcript.py --search "<query>" --max 5 --json-out` to find videos
- If yt-dlp is not installed, use WebSearch to find relevant YouTube videos on the topic
- Present the found videos and ask which ones to process (or process all)

### Step 2: Extract Transcripts

For each video:
1. Run: `python D:/Marketing/scripts/yt_transcript.py "<video_url>" --vault "D:/Marketing/vault" --json-out`
2. If the Python script fails (missing dependencies), fall back to:
   - Use WebFetch on `https://www.youtube.com/watch?v=<VIDEO_ID>` to get the page
   - Extract what information you can
   - Note: For best results, install dependencies: `pip install youtube-transcript-api yt-dlp`

### Step 3: Read and Process the Saved Notes

After transcripts are saved to `D:/Marketing/vault/00-Inbox/`:
1. Read each saved transcript file
2. Create a research summary in `D:/Marketing/vault/01-Research/` with:

```markdown
---
title: "Research: <Topic>"
date: <today>
sources: [list of video URLs]
tags: [research, youtube, <topic-tags>]
type: research
status: active
---

# Research: <Topic>

## Key Findings
- Bullet points of the most important insights across all videos

## Video Summaries

### <Video 1 Title>
- **Channel:** <channel>
- **Key Points:**
  - Point 1
  - Point 2
- **Notable Quotes:** (paraphrased)
- **Relevance:** Why this matters

### <Video 2 Title>
...

## Cross-References
- Common themes across videos
- Contradictions or debates
- Gaps in coverage

## Action Items
- [ ] What to do with this research
- [ ] Follow-up topics to explore

## Related Notes
- [[links to other vault notes if relevant]]
```

### Step 4: Report Back

Tell the user:
- How many videos processed
- Where files are saved
- Key findings summary
- Suggest running `/yt-prompts` to generate prompts from this research
- Suggest running `/yt-knowledge` to build knowledge base entries

## Important Notes

- Always save raw transcripts to `00-Inbox/`
- Always save research summaries to `01-Research/`
- Use Obsidian-compatible markdown (wikilinks, frontmatter, callouts)
- Tag everything for easy searching
- If transcript extraction fails for a video, note it and continue with others
