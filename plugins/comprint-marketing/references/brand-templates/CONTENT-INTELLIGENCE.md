# Content Intelligence Engine

> **Version:** 2.0
> **Purpose:** Universal framework reference for content generation. Read by brand-generate, scripter, and repurpose skills.
> **Shared across all brands.** Brand-specific adaptation happens via operator selection + personality scores in VOICE.md.

---

## How This File Works

This file contains 6 technique categories + an interactive options menu. Before generating content, the operator is presented with **2 creative decisions** (Hook Approach + Viral Target). The operator picks their direction, then the engine applies the matching techniques automatically.

If the operator says **"surprise me"** or skips selection, the system falls back to score-based defaults using VOICE.md personality scores.

**The golden rule:** Frameworks provide STRUCTURE. VOICE.md provides WORDS. If a technique sounds off-brand when filtered through the brand's voice, rewrite the technique to fit the voice — never the other way around.

---

## Category 1: Hook Engineering

Every piece of content lives or dies in the first 3 seconds (video) or first line (text). These are the rules for hooks that stop the scroll.

### Core Principles
- Lead with the dream OUTCOME, never features or setup
- Pattern interrupts break expectations — say something they didn't expect
- Bold specific claims always include numbers ("347 posts", "2 hours", "$3,000/month")
- Curiosity gaps create tension that demands resolution — open a loop the audience MUST close
- Never open with context or backstory — hook first, context second

### Hook Formula Bank

Use these fill-in-the-blank formulas. Adapt phrasing to match the brand's voice.

| # | Formula | Best For |
|---|---------|----------|
| H1 | "I [impressive result] in [timeframe]. Here's the [number]-step system:" | Authority brands, proof-driven content |
| H2 | "Everyone's doing [common thing] wrong. This one change [specific result]:" | Contrarian takes, pattern interrupts |
| H3 | "I studied [number] [things]. Found [number] pattern(s). It's stupid simple:" | Data-driven, research-backed content |
| H4 | "[Number] [things] that [dream outcome] (most people only know #[N]):" | Listicles, carousels, educational content |
| H5 | "Stop [common mistake]. Do this instead:" | Quick tips, reels, problem-aware audiences |
| H6 | "The [number]-second trick that [specific outcome]:" | Short reels, quick wins |
| H7 | "Nobody's talking about this, but [insider knowledge]:" | Insider content, niche expertise |
| H8 | "I was [relatable struggle]. Then I discovered [epiphany]:" | Story-driven, vulnerable brands |
| H9 | "Here's why [controversial take] (with proof):" | Bold brands, thought leadership |
| H10 | "[Dream outcome] without [main objection]. Here's how:" | Offer-adjacent, solution-aware audiences |
| H11 | "What [number] [successful people] do differently about [topic]:" | Aspirational, authority content |
| H12 | "The [audience] who [dream outcome] all do this ONE thing:" | Identity-driven, community content |

### Hook Approach Options (present to operator before generating)

Present these 4 approaches in plain language. The operator picks one BEFORE content is generated.

| Option | What The Operator Sees | Maps To Formulas |
|--------|----------------------|-----------------|
| **A) Proof-Led** | Open with specific data, results, or bold claims backed by evidence. *Example: "I analyzed 247 AI tools. Only 3 are worth your money."* | H1, H3, H9, H11 |
| **B) Pattern Interrupt** | Flip expectations, challenge common beliefs, surprise them. *Example: "Stop posting carousels. Here's why they're killing your reach."* | H2, H5, H6 |
| **C) Story-Driven** | Lead with a relatable struggle, journey, or transformation. *Example: "I was mass-DMing 50 people a day. Then I discovered this."* | H8, H12, H4 |
| **D) Curiosity Gap** | Tease insider knowledge or counterintuitive insights. *Example: "Nobody's talking about this Instagram update. But it changes everything."* | H7, H4, H9 |

**When presenting options:** Replace the example hooks with niche-specific examples from the brand's CONTENT.md pillars. The examples above are generic — generate real ones from the brand context.

### "Surprise Me" Defaults (score-based fallback)

If the operator says "surprise me", skips selection, or doesn't specify, auto-pick based on VOICE.md:

| Brand Personality Score | Default To |
|------------------------|-----------|
| Authority >= 4 | A) Proof-Led |
| Humor >= 4 | B) Pattern Interrupt |
| Vulnerability >= 3 OR Warmth >= 4 | C) Story-Driven |
| Technical Depth >= 4 | D) Curiosity Gap |
| Ambition >= 4 | A) Proof-Led |
| No strong score (all < 4) | A) Proof-Led (safest default) |

When using defaults, announce the choice: *"Going with **Proof-Led** based on your brand profile."*

### Hook Quality Gate
Every hook must pass ALL of these:
- [ ] Contains at least ONE specific number, outcome, or timeframe
- [ ] Would make someone stop mid-scroll (not just nod along)
- [ ] Does NOT start with "In this post..." or "Today I want to talk about..."
- [ ] Matches the brand's voice when read aloud
- [ ] Creates a curiosity gap or makes a promise the content delivers on

---

## Viral Target Options (present to operator before generating)

Present these alongside the Hook Approach options. The operator picks ONE viral optimization target.

| Option | What The Operator Sees | Algorithm Signal |
|--------|----------------------|-----------------|
| **A) Shares** | Make it so relatable or useful they forward it to someone. DM-worthy. | Relatability, humor, "tag someone", identity content |
| **B) Saves** | Pack it with frameworks, checklists, reference-worthy value. Bookmark-worthy. | Step-by-step, templates, numbered lists, cheat sheets |
| **C) Watch-time** | Build story tension and open loops that keep them watching to the end. | Cliffhangers, "wait for it", payoff structure, pattern stacking |

### "Surprise Me" Defaults

If the operator skips viral target selection:

| Content Type | Default Viral Target |
|-------------|---------------------|
| Carousel | Always B) Saves |
| Reel + Humor >= 4 | A) Shares |
| Reel + Story-Driven hook chosen | C) Watch-time |
| Reel (default) | A) Shares |
| LinkedIn | A) Shares |
| Story | A) Shares |

When using defaults, announce: *"Optimizing for **Saves** (best for carousels)."*

### Batch "Mixed" Option
For batch generation (3+ pieces), offer an additional option:
- **"Mixed"** — Vary the approach across pieces for audience variety
- If "mixed", distribute using score-based weighting (strongest personality score gets most pieces)
- Announce the distribution: *"Going with 2 Proof-Led + 1 Story-Driven, mixed across Shares and Saves."*

---

## Category 2: Story Architecture

Every content piece tells a story — even a 15-second reel. The structure changes by format, but the principle is the same: tension creates attention.

### Universal Micro-Story Arc
**Struggle --> Wall --> Epiphany --> Framework --> Transformation**

This arc compresses or expands based on content length:

### Reel Story Beats (mapped to 4-Block System)

| Block | Timing | Story Beat | What Happens |
|-------|--------|-----------|-------------|
| HOOK | 0:00-0:03 | Struggle OR Dream Outcome | Pattern interrupt. Open with the pain point or the end result — never setup. |
| PULL-BACK | 0:03-0:10 | The Wall | What went wrong. Why the common approach fails. Why this matters. Build tension. |
| MEAT | 0:10-0:35 | Epiphany + Framework | The breakthrough moment. Present the system/method/steps. Numbered frameworks preferred. This is the actual value. |
| CLOSE | 0:35-0:55 | Transformation + CTA | Show the result. Before/after. Specific outcome. Then CTA with conversion trigger. |

For short reels (7-15 sec): Compress to HOOK + MEAT + CTA (skip PULL-BACK).

### Carousel Narrative Flow

| Slide | Purpose | Rule |
|-------|---------|------|
| 1 | Hook | Bold claim or question. Stop the scroll. ONE idea, large text. |
| 2-3 | Problem Agitation | Make them FEEL the problem. "You've tried X. It didn't work because..." |
| 4-7 | Framework / Solution | One idea per slide. Specific, actionable. Numbered steps if possible. |
| 8 | Transformation Proof | Before/after, specific numbers, or testimonial quote. |
| 9-10 | CTA + Value Stack | What to do next. Stack value. Include save/share trigger. |

### LinkedIn Long-Form Structure

```
Line 1-2: Hook (must work BEFORE "see more" truncation — this is everything)

[blank line]

Short paragraph: Personal story or case study opening (3-4 lines max)

[blank line]

The wall / the problem (2-3 lines)

[blank line]

The epiphany (1-2 lines — the turning point)

[blank line]

Framework with numbered steps:
1. [Step] — [one-line explanation]
2. [Step] — [one-line explanation]
3. [Step] — [one-line explanation]

[blank line]

Vulnerable insight or contrarian close (1-2 lines)

[blank line]

CTA as engagement question: "[Specific question that invites opinion/experience]"

[hashtags at very end, 3-5 max]
```

### Caption Micro-Story
For standalone captions (photos, static posts):

**Situation --> Twist --> Lesson --> CTA** (4 segments, each 1-2 lines max)

---

## Category 3: Awareness Calibration

Different audiences need different messages. The audience's awareness level (from AUDIENCE.md) determines what to say — and what NOT to say.

### Awareness Level --> Message Strategy

| Level | What They Know | Your Message Strategy | Hook Style | CTA Style |
|-------|---------------|----------------------|-----------|----------|
| **Unaware** | Don't know they have a problem | Agitate the problem. Make them feel it. "Did you know..." / "Most [audience] don't realize..." | Problem-first hooks (H5, H7) | Soft — "Follow for more" |
| **Problem Aware** | Know the problem, not solutions | Paint the dream outcome. Show life AFTER. "Imagine if..." / "What if you could..." | Dream-outcome hooks (H1, H10) | Medium — "Save this" |
| **Solution Aware** | Know solutions exist, not yours | Differentiate YOUR approach. "Everyone says [X]. Here's why that's wrong:" / "The [your method] difference:" | Contrarian hooks (H2, H9) | Medium — "Link in bio" |
| **Product Aware** | Know your product, haven't bought | Stack proof and reduce risk. Testimonials, case studies, guarantees. "Here's what happened when [client] tried this:" | Proof hooks (H1, H3, H11) | Strong — Value stack + CTA |
| **Most Aware** | Past customers | New offers, exclusivity, urgency. Give them a reason to act NOW. | Urgency hooks (H6, H9) | Direct — Deadline + bonus |

### Batch Distribution Rule
When generating a batch of content, distribute across awareness levels:
- 60% targeting the brand's PRIMARY awareness level (from AUDIENCE.md)
- 20% targeting ONE level above (moving people down the funnel)
- 20% targeting ONE level below (catching new audience)

### Default
If AUDIENCE.md doesn't specify awareness distribution, assume **Problem Aware** — the largest segment for most growing brands.

---

## Category 4: Conversion Triggers

Every piece of content has a goal. These triggers turn attention into action — without being sleazy.

### Specificity Rule
Specificity ALWAYS beats vagueness. Replace:
- "save time" --> "save 2 hours per day"
- "grow your business" --> "add $5K/month in recurring revenue"
- "get more followers" --> "gain 500 followers in 30 days"
- "improve your content" --> "double your engagement rate in 2 weeks"

### Value Stacking
Before ANY price mention or purchase CTA:
1. List what they get (3-5 concrete items)
2. Attach perceived value to each item
3. Show total value vs. what they actually pay/do
4. Make the gap feel absurd ("$2,000 in value for $0 — just follow")

### Risk Reversal
Remove the fear of taking action:
- "Try it free for [timeframe]"
- "If you don't [specific result] in [timeframe], I'll [specific remedy]"
- "No commitment, cancel anytime, keep everything"
- "100% money-back guarantee — no questions asked"

### Urgency (with reason-why — never fake scarcity)
- "Closing [day] because I can only onboard [number] clients per [timeframe]"
- "Price goes up after launch week — here's why: [genuine reason]"
- "Limited to [number] spots because [genuine operational constraint]"
- NEVER: "Only 3 left!" when there's unlimited supply
- NEVER: Fake countdown timers or manufactured scarcity

### Social Proof Placement
- Before CTA: "[Number] [people] already [doing this / using this / seeing results]"
- After a bold claim: "([Client name] went from [before metric] to [after metric] in [timeframe])"
- As the hook itself: "Why [number] [people] switched to [this approach] last [timeframe]"

### Level-Gated CTA Intensity

| Brand Level | CTA Intensity | Examples |
|------------|--------------|---------|
| **L1 (0-500)** | Soft — value first, no urgency, no price | "Save this for later" / "Follow for more [topic]" / "DM me [keyword]" |
| **L2 (500-2K)** | Medium — social proof where available, light urgency | "Link in bio" / "Comment [word] and I'll send the guide" / "[Number] people already have this" |
| **L3 (2K+)** | Full — proof + urgency + guarantee + clear action | Value stack + deadline + risk reversal + specific next step |

### Offer Status Gates

| OFFER.md Status | What You Can Do | What You Cannot Do |
|----------------|----------------|-------------------|
| DRAFT | Reference the problem you solve. Soft CTAs only. | Mention price, specific offer details, or guarantees |
| DEFINED | Reference offer with soft CTA. Use one-liner. | Use full urgency, testimonials, or value stacks |
| TESTED | Full conversion triggers. Reference specific results. | Use unverified testimonials or fabricated proof |
| PROVEN | Everything. Testimonials, case studies, full value stack, urgency. | Nothing — full toolkit available |

---

## Category 5: Viral Mechanics

Each content type optimizes for ONE primary algorithm signal. Pick the right viral mechanic for the right format.

### Save Triggers (Carousels, Educational Content)
Content people bookmark to reference later:
- Include checklists, templates, or step-by-step frameworks
- Use "Save this for when you need it" or "Bookmark this [topic] cheat sheet"
- Make content REFERENCE-WORTHY — something they'll come back to, not just consume once
- Numbered lists and how-to formats have the highest save rates

### Share Triggers (Reels, Humor, Relatable Content)
Content people send to friends via DM:
- Relatability — "Tag someone who does this"
- Identity — "Only [audience type] will understand this"
- Usefulness — "Send this to someone who needs to hear it"
- Humor — content that makes people think "my friend/coworker needs to see this"

### Comment Triggers (LinkedIn, Engagement-Focused Content)
Content that starts conversations:
- Specific question CTAs — "Which one surprised you most?" / "What would you add?"
- Opinion requests — "Agree or disagree? Drop your take below"
- Fill-in-the-blank — "My biggest [topic] mistake was ___"
- Hot takes — make a claim people want to debate
- **NEVER** use "Thoughts?" — too generic, gets zero engagement

### Watch-Time Triggers (Reels, YouTube)
Content that keeps people watching:
- Open loops — start a story, don't resolve it until the end
- "Wait for it..." moments — build anticipation for a payoff
- Pattern stacking — deliver value so consistently they don't want to leave
- Cliffhanger transitions between story blocks

### Engagement Bait Quality Rules
- Must feel natural, not manipulative
- Must relate to the content topic (not random "comment your zodiac sign")
- Must give the audience something to react TO — a claim, a ranking, a comparison, a hot take
- If it would feel cringe reading it out loud, rewrite it

---

## Category 6: Platform Execution

Each platform has different rules. Content must be NATIVE to the platform — never cross-posted identically.

### Instagram
- Hook in first 3 seconds (visual payoff, not setup)
- 5 hashtags maximum (more reduces reach since Dec 2025)
- Share to Stories within 1 hour of posting (with engagement sticker)
- Reply to ALL comments within 1 hour (golden window)
- Reels: 9:16, 7-30 sec sweet spot, sound on
- Carousels: 4:5, 8-12 slides, save-optimized

### LinkedIn
- First 2 lines must hook BEFORE "see more" truncation
- Dwell-time optimized — use whitespace, short paragraphs, easy scanning
- Links go in FIRST COMMENT, never in post body (suppresses reach)
- 3-5 hashtags at the END of the post
- Never edit within 4 hours (resets algorithm distribution)
- Max 1 post per day (more = cannibalization)
- Native video only — never YouTube links

### YouTube
- Thumbnail = 50% of performance (face + text + context, max 3 elements)
- Hook in first 2 seconds ("By the end of this video, you'll know exactly how to...")
- End screens ALWAYS (keeps viewers on platform = higher session time)
- 5-10 tags per video (mix broad + specific, title is most important for SEO)
- Shorts: 9:16, under 60 sec, hook in 1 sec
- Long-form: 16:9, 8-15 min optimal for ad revenue + watch time

### Cross-Platform Rules
- NEVER post the same content on all platforms the same day
- Stagger across 2-3 days (IG Monday, LI Wednesday, YT Friday)
- Adapt TONE per platform: IG = casual/visual, LI = professional/narrative, YT = educational/detailed
- Each platform gets a DIFFERENT hook for the same topic
- Repurpose the IDEA, not the exact words

---

## Content Output Standards

Every piece of content generated using this intelligence engine must:

- [ ] Hook in first 3 seconds / first sentence (passes Hook Engineering quality gate)
- [ ] Include at least ONE specific number, metric, or timeframe
- [ ] Follow Story Architecture for its content type
- [ ] Match the audience's awareness level (from Awareness Calibration)
- [ ] End with a clear CTA (level-gated via Conversion Triggers)
- [ ] Include ONE viral mechanic trigger (save / share / comment / watch-time)
- [ ] Be platform-native (follows Platform Execution rules)
- [ ] Pass the brand's voice compliance scoring (VOICE.md always wins)

### NEVER
- Use vague language ("might," "could," "possibly," "maybe")
- Write generic advice without specific examples or numbers
- Create content without a CTA
- Ignore the audience awareness level
- Use fake urgency or manufactured scarcity
- Sound like a different brand (frameworks provide structure, not personality)
