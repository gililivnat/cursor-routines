---
name: create-presentation
description: Create presentation decks in the user's signature style. Multi-step workflow covering brief, data gathering, outline, creation (interactive HTML with charts, diagrams, animations), and sharing via GitHub Pages. Use when the user asks to create a presentation, deck, slides, or pitch.
---

# Create Presentation

Design, build, and share presentations as interactive HTML slide decks in the user's minimalist style.

## When to Use

Run this skill when asked to:
- Create a presentation, deck, or slides
- Build a pitch or update for stakeholders
- Prepare visual material for a meeting or review

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

### Step 1: Brief and alignment

Before anything else, confirm these with the user:

| Question | Why it matters |
|----------|---------------|
| **Audience** | Exec, team, customer, external, conference? Determines depth and tone |
| **Goal** | Inform, persuade, align, inspire, or request a decision? |
| **Context** | Meeting, async review, conference talk, internal share? |
| **Length** | Approximate slide count or time limit |
| **Key messages** | What are the 2-3 things the audience must walk away with? |
| **Data needs** | What metrics or data should be included? Any specific charts? |
| **Diagrams** | Any flowcharts, journeys, or architecture diagrams needed? |
| **Materials** | Any existing docs, research, or data the user wants to incorporate? |

Collect all answers before proceeding.

### Step 2: Data gathering

Based on the brief, identify what data is needed and query for it.

**Data sources:**

| Source | When to use |
|--------|-------------|
| Data Warehouse MCP | Product metrics, usage data, funnel analysis. Use your SQL query tool for specific queries, your data exploration tool for exploration |
| BigBrain MCP (`user-bigbrain-mcp`) | Broader analytics, error analysis, PR data |
| your input | Context, qualitative data, specific numbers he provides |

**Process:**
1. Read `your data guide` for current table schemas and metric definitions
2. Draft the queries needed based on the brief
3. Execute queries and aggregate results
4. Present a data summary to the user: what was found, what's interesting, what could go on slides
5. the user confirms which data points to include and which to cut
6. If no data is needed (e.g. a strategy presentation), skip this step

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

**Technical reference:**
- Read the template at `skills/system/presentation-html-template.md` for the full CSS, JS, and component library
- Read `your tone of voice guide` before writing any slide content

**Build process:**
1. Start in browser canvas for live preview during development
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
- Save the final HTML to `Output/presentations/YYYY-MM-DD-topic-name/index.html`
- Open with `open` (macOS) for final verification

**Optional: exec summary:**
- If requested, generate a condensed 1-page exec summary
- Structure: key takeaways (3-5 bullets) + one key chart/metric + next steps
- Save as `Output/presentations/YYYY-MM-DD-topic-name/exec-summary.html`
- Use the routine HTML template CSS (`skills/system/routine-html-template.md`) for the exec summary
- Present for review before finalising

**Sharing via GitHub Pages:**
- Default: push to `giladli-mond` GitHub account (ask the user if unsure about audience/repo)
- Push the presentation folder to the presentations GitHub Pages repo
- Share the URL with the audience

## Output

| Output | Location | Format |
|--------|----------|--------|
| Presentation | `Output/presentations/YYYY-MM-DD-topic-name/index.html` | Standalone HTML |
| Exec summary | `Output/presentations/YYYY-MM-DD-topic-name/exec-summary.html` | Standalone HTML (print-ready) |
| Shared URL | GitHub Pages | Link |

## Tone on Slides

Before writing any slide content, read `your tone of voice guide` for the user's full tone of voice profile. Adapt it for the presentation audience.

Match the user's voice: calm, confident, clear. No hyperbole. No filler words.
Avoid: "Revolutionary", "Game-changing", "Exciting", "Synergy"
Prefer: Specific, evidence-backed, understated confidence.

## Additional Resources

- HTML template: `skills/system/presentation-html-template.md`
- Routine HTML template (for exec summaries): `skills/system/routine-html-template.md`
- Storytelling agent: `Agents/storytelling-agent.md`
- Craft narrative skill: `skills/craft-narrative/SKILL.md`
- Tone of voice: `your tone of voice guide`
- Human writing: `~/.cursor/skills/make-human-lite/SKILL.md`
- Data guide: `your data guide`
- Product strategy: `your product strategy documentation`

## Change Log

| Date | Change |
|------|--------|
| Mar 2026 | Initial version. Basic outline workflow with design system. |
| 17 Mar 2026 | Major upgrade. Full multi-step workflow: brief, data gathering, outline, interactive HTML creation, sharing via GitHub Pages. Added Chart.js with zoom/pan, SVG diagram generation, speaker notes, element-level animations, exec summary support. |
| 26 Mar 2026 | Added Step 3b: Storytelling agent integration. After outline approval, the `craft-narrative` skill shapes the outline into a compelling story arc before creation begins. |
