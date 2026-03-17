---
description: "Quick Instagram profile or hashtag analysis - analyze any public account or hashtag and save to vault"
---

# Instagram Quick Research

**Input:** $ARGUMENTS

## Workflow

### If the input is a username (@something):
1. Run: `python D:/Marketing/scripts/ig_analyzer.py "<username>" --vault "D:/Marketing/vault" --json-out`
2. Read the saved file from `D:/Marketing/vault/00-Inbox/`
3. Provide a quick analysis:
   - Account health (engagement rate vs niche benchmarks)
   - Content mix assessment
   - Best performing content type
   - 3 quick improvement suggestions
4. Suggest running `/ig-ai` for full competitive analysis

### If the input is a hashtag (#something):
1. Run: `python D:/Marketing/scripts/ig_analyzer.py --hashtag "<tag>" --vault "D:/Marketing/vault" --json-out`
2. Read the saved file
3. Analyze:
   - Is this hashtag worth using? (size, competition, relevance)
   - Who dominates this hashtag?
   - Suggest related hashtags
4. Suggest running `/ig-ai` for full hashtag strategy

### If the input is multiple usernames:
1. Run: `python D:/Marketing/scripts/ig_analyzer.py --batch "user1,user2,user3" --vault "D:/Marketing/vault" --json-out`
2. Read all saved files
3. Create a quick comparison table
4. Identify who's winning and why

### If the script fails:
- Fall back to WebSearch for the profile data
- Note the limitation and suggest retrying later
- Still provide analysis based on available web data

## Output
Always save to vault AND provide a concise verbal summary to the user.
