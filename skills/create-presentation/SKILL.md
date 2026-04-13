---
name: create-presentation
description: Create presentation decks in the user's signature style. Default whenever the user asks to work on a presentation, deck, slides, or pitch. Multi-step workflow (brief, research, approved skeleton, build) with interactive HTML, charts, diagrams, animations, and optional GitHub Pages. Exception — team biweekly/monthly/quarterly decks in Tools/campaigns-decks/ use that repo's slides-* skills instead.
---

# Create Presentation

Design, build, and share presentations as interactive HTML slide decks in the user's minimalist style.

## When to Use

**Default:** Run this skill whenever the user asks to work on a presentation or deck (any topic or session), including:
- Create or revise a presentation, deck, or slides
- Build a pitch or stakeholder update
- Prepare visual material for a meeting, training, or review

**Exception:** Work on recurring **team** decks **inside** `Tools/campaigns-decks/` (Next.js biweekly / monthly / quarterly) uses `Tools/campaigns-decks/AGENTS.md` and the `slides-*` skills, not this workflow.

## End-to-end SOP

**Read first:** `Tools/Workflows/create-presentation.md` — phased flow (**Research → Skeleton → Build**), including **web + internal** research, skeleton approval gate, and **all-at-once vs slide-by-slide** build modes. This skill holds the design system, template paths, and detailed steps; the workflow is the checklist order.

**Save path:** `Output/<project-name>/presentation/` (one folder per project under `Output/`, then `presentation/` for the deck). Never `Output/presentations/`.

## Design System (v1)

| Element | Value |
|---------|-------|
| Background | Black (`#0a0a0a`) |
| Primary text | White (`#ffffff`) |
| Accent | Cyan (`#00d4ff`) |
| Secondary text | Light grey (`#a0a0a0`) |
| Muted text | Dark grey (`#555555`) |
| Positive | Green (`#4ade80`) |
| Negative | Red (`#f87171`) |
| Warning | Amber (`#fbbf24`) |
| Font | Inter (Google Fonts CDN) |
| Layout | Minimalist, generous whitespace, never crowded |
| Animations | Element-level: fade-up, fade-in, scale-in, slide-left |
| Charts | Chart.js with zoom/pan plugin, interactive tooltips |
| Diagrams | SVG generated from descriptions |
| Navigation | Arrow keys, minimal progress bar at bottom |

This is v1. The design system will be refined over time as more presentations are created. The goal is to settle on 1-2 polished themes, stored as CSS presets.

## Guiding Principles

- **Less is more.** Every word on a slide must earn its place
- **One idea per slide.** Two ideas = two slides
- **Data speaks visually.** Charts over tables. Annotate the key insight in cyan
- **Hierarchy is clarity.** Large headline, supporting detail, source (small, muted)
- **Easy on the eye.** Dark background, let whitespace breathe
- **Interactive where possible.** Charts respond to hover, zoom, and click

## Workflow

### Step 0: Read memory

Read `your learnings file`, `your decisions file`, and `Memory/feedback.md` before starting. These contain presentation preferences, past decisions, and feedback on previous decks.

### Step 1: Brief and alignment

**1a — Audience and length first.** Do not start Step 2 (research, SQL, web) until **audience** and **length** are clear:

| Question | Why it matters |
|----------|---------------|
| **Audience** | Who is in the room, what they know, what they need — sets depth, tone, jargon vs plain language |
| **Length** | Time slot and/or slide budget — sets how many ideas fit and pacing |

If unclear, ask. Summarise in one line before moving on.

**1b — Rest of the brief.** Then confirm:

| Question | Why it matters |
|----------|---------------|
| **Goal** | Inform, persuade, align, inspire, or request a decision? |
| **Context** | Meeting, async review, conference talk, internal share? |
| **Key messages** | What are the 2-3 things the audience must walk away with? |
| **Data needs** | What metrics or data should be included? Any specific charts? |
| **Diagrams** | Any flowcharts, journeys, or architecture diagrams needed? |
| **Materials** | Any existing docs, research, or data the user wants to incorporate? |

Collect **1a** before any research; collect **1b** before locking the skeleton.

### Step 2: Research and data gathering

Based on the brief, identify what evidence is needed and collect it **internally and externally** (see `Tools/Workflows/create-presentation.md` Phase 1).

**Internal data sources:**

| Source | When to use |
|--------|-------------|
| Data Warehouse MCP | Product metrics, usage data, funnel analysis. Use your SQL query tool for specific queries, your data exploration tool for exploration |
| Analytics MCP | Broader analytics, error analysis, PR data |
| `Strategy/`, `your product strategy documentation` | Strategic framing for product-related storylines |
| Notion MCP (`user-notion`) | Projects or notes when the deck is tied to documented work |
| Slack MCP (your Slack MCP) | Recent threads only when the deck materially depends on them |
| `Output/` | Prior artefacts to reuse or cite |
| your input | Context, qualitative data, specific numbers, constraints |

**Web and external:**

| Method | When to use |
|--------|-------------|
| Web search | Competitor moves, market stats, benchmarks, definitions |
| Fetch URL | Pages or articles the user links |
| Public docs | e.g. your product help content when relevant |

Cite external sources in the research summary (title, URL, date if known).

**Process:**
1. Read `your data guide` for current table schemas and metric definitions before SQL
2. Draft queries or searches based on the brief; execute and aggregate
3. Present a **research summary** to the user: findings, gaps, implications for slides
4. the user confirms what to include, cut, or verify
5. If no research is needed (e.g. a short internal update), skip accordingly

### Step 3: Outline

Create a structured slide-by-slide outline. Use this default narrative arc unless the user specifies a different structure:

1. **Title slide** — topic name + context line + date
2. **The situation** — what's the problem, opportunity, or context?
3. **The insight** — what do we know, believe, or have discovered?
4. **The proposal** — what are we doing or recommending?
5. **Evidence** — data, charts, metrics, examples
6. **Next steps** — clear action or ask
7. **Appendix** (optional) — supporting detail

For each slide in the outline, specify:

```
## Slide N: [Title]
**Type:** content / stat / chart / diagram / cards / two-col / quote
**Headline:** [bold, max 8 words]
**Body:**
- [bullet or description]
**Visual:** [chart type + data, or diagram description, or none]
**Animation:** [e.g. "bullets fade-up staggered", "chart scale-in"]
**Notes:** [speaker notes for this slide]
```

Review the outline with the user in conversation. Iterate until the skeleton is approved.

### Step 3b: Narrative shaping (Storytelling Agent)

Once the outline is approved, invoke the Storytelling agent to shape it into a compelling narrative.

1. Load the `craft-narrative` skill at `skills/craft-narrative/SKILL.md`
2. Pass the approved outline, audience, goal, and key messages to the skill
3. The Storytelling agent will:
   - Choose the best story structure (SCR, What/So What/Now What, Problem/Insight/Solution, etc.)
   - Define story beats for each slide (opening hook, tension, evidence, resolution)
   - Recommend data placement (introduce charts when the audience is asking "is this real?")
   - Write transition logic between slides (the invisible narrative thread)
   - Draft speaker notes with anchor phrases for key moments
   - Suggest 2-3 opening hook options
   - Recommend a closing approach
   - Flag any content that should be cut or moved to appendix
4. Review the narrative with the user. Iterate until the story arc feels right
5. Proceed to Step 4 with the narrative-enriched outline

**Note:** This step is strongly recommended but can be skipped if the user explicitly asks to go straight to creation.

### Step 4: Creation

Build the presentation as a live interactive HTML deck.

**Build modes** (the user chooses; see workflow Phase 3):

| Mode | When |
|------|------|
| **All at once** | Skeleton is stable; consistent layouts; tight time — build full `index.html`, then polish |
| **Slide by slide** (or batches of 3–5) | Heavy visuals, evolving charts, or the user wants incremental review — **required:** preview in **Cursor’s Browser** beside `index.html` (see **Preview in Cursor** below) |

**Preview in Cursor (slide-by-slide / batches):** Keep the deck visible in the IDE, not only in chat.

1. Open **Cursor’s Browser** (Command Palette → Browser / Open Browser) or the integrated preview your build offers. Split view: editor + Browser.
2. Load the deck: open `index.html` in the Browser, or run a static server in the presentation folder (`npx --yes serve . -p 3456`) and open `http://localhost:3456` if `file://` blocks scripts.
3. After each slide or batch: save, refresh the Browser.

**Technical reference:**
- Read the template at `skills/system/presentation-html-template.md` for the full CSS, JS, and component library
- Read `your tone of voice guide` before writing any slide content

**Build process:**
1. For slide-by-slide or batches: set up **Cursor’s Browser** (or Simple Browser) beside the editor before adding slides (see **Preview in Cursor** above). For all-at-once, still recommended for polish.
2. Build the HTML using the template's base structure
3. For each slide, use the appropriate component pattern from the template
4. Generate charts using Chart.js with the template's styling defaults
5. Generate diagrams as SVGs from the user's descriptions
6. Add speaker notes as `<aside class="notes">` in each slide
7. Apply element-level animations with staggered delays
8. Apply `make-human-lite` to all text content before finalising

**Chart capabilities:**
- All charts include hover tooltips with detailed values
- Scroll-to-zoom and click-and-drag pan enabled via `chartjs-plugin-zoom`
- Click to toggle dataset visibility (Chart.js built-in)
- Use the colour palette from the template's Chart Styling Defaults section

**Diagram generation:**
- the user describes what the diagram should show
- Generate SVG following the template's diagram styling rules
- Alternatively, use Mermaid for quick diagrams

**Navigation:**
- Arrow keys (left/right and up/down) to move between slides
- `N` key to toggle speaker notes panel
- Minimal progress bar at bottom
- Subtle slide counter (bottom-right)

**Iteration:**
- After the first build, review with the user in conversation
- Adjust individual slides as requested
- Continue iterating until the user is satisfied

### Step 5: Finalise and share

**Save:**
- Save the final HTML to `Output/<project-name>/presentation/index.html` (see workflow for project folder naming)
- Open with `open` (macOS) for final verification

**Optional: exec summary:**
- If requested, generate a condensed 1-page exec summary
- Structure: key takeaways (3-5 bullets) + one key chart/metric + next steps
- Save as `Output/<project-name>/presentation/exec-summary.html`
- Use the routine HTML template CSS (`skills/system/routine-html-template.md`) for the exec summary
- Present for review before finalising

**Sharing via GitHub Pages:**
- Default: push to `your-github-account` GitHub account (ask the user if unsure about audience/repo)
- Push the presentation folder to the presentations GitHub Pages repo
- Share the URL with the audience

## Output

| Output | Location | Format |
|--------|----------|--------|
| Presentation | `Output/<project-name>/presentation/index.html` | Standalone HTML |
| Exec summary | `Output/<project-name>/presentation/exec-summary.html` | Standalone HTML (print-ready) |
| Shared URL | GitHub Pages | Link |

## Tone on Slides

Before writing any slide content, read `your tone of voice guide` for the user's full tone of voice profile. Adapt it for the presentation audience.

Match the user's voice: calm, confident, clear. No hyperbole. No filler words.
Avoid: "Revolutionary", "Game-changing", "Exciting", "Synergy"
Prefer: Specific, evidence-backed, understated confidence.

## After Approval

When the user approves the presentation:
1. Update `Memory/feedback.md` with what worked and what needed changing
2. Update `your learnings file` if new preferences were discovered (e.g. layout, animation, data visualisation)
3. Update `your decisions file` if meaningful decisions were made (e.g. design system evolution, new template patterns)

## Additional Resources

- **Workflow (SOP):** `Tools/Workflows/create-presentation.md`
- HTML template: `skills/system/presentation-html-template.md`
- Routine HTML template (for exec summaries): `skills/system/routine-html-template.md`
- Storytelling agent: `Agents/storytelling-agent.md`
- Craft narrative skill: `skills/craft-narrative/SKILL.md`
- Tone of voice: `your tone of voice guide`
- Human writing: `~/.cursor/skills/make-human-lite/SKILL.md`
- Data guide: `your data guide`
- Product strategy: `your product strategy documentation`
- Memory: `your learnings file`, `your decisions file`, `Memory/feedback.md`

## Change Log

| Date | Change |
|------|--------|
| Mar 2026 | Initial version. Basic outline workflow with design system. |
| 17 Mar 2026 | Major upgrade. Full multi-step workflow: brief, data gathering, outline, interactive HTML creation, sharing via GitHub Pages. Added Chart.js with zoom/pan, SVG diagram generation, speaker notes, element-level animations, exec summary support. |
| 26 Mar 2026 | Added Step 3b: Storytelling agent integration. After outline approval, the `craft-narrative` skill shapes the outline into a compelling story arc before creation begins. |
| 8 Apr 2026 | Linked `Tools/Workflows/create-presentation.md`. Expanded Step 2 to web + internal research; Step 4 build modes (all-at-once vs slide-by-slide). |
| 8 Apr 2026 | Scope: default for **any** deck or presentation request; **`Tools/campaigns-decks/`** team recurring decks are the exception (`slides-*`). |
| 9 Apr 2026 | Save path: **`Output/<project-name>/presentation/`** per workflow (replaces `Output/presentations/YYYY-MM-DD-topic-name/`). |
| 8 Apr 2026 | Step 1 split: audience + length (1a) before research; remainder of brief (1b). |
| 8 Apr 2026 | Slide-by-slide: Cursor’s Browser beside editor; local `serve` if `file://` blocks scripts. |
