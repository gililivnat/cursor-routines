---
name: plan-quarter
description: Supports quarterly planning by reviewing current quarter progress, summarising themes, and drafting priorities for the next quarter. Use during the last month of each quarter.
---

# Plan Quarter

## When to Use

This skill is triggered during the **last month of each quarter** as part of the weekly routine. It can also be run on demand when asked to start quarterly planning.

This is a thinking aid and planning scaffold, not a fully automated task.

## Inputs

- Current quarter's OKRs (if documented)
- Weekly summaries from `your output folder/` across the quarter
- Product strategy from `your product strategy documentation`
- Competitor analysis reports from `your output folder/`
- Scheduled tasks and their completion status from `your scheduled tasks file`
- Memory: `your learnings file` and `your decisions file`

## Tools

### Data Warehouse MCP

| Tool | Purpose |
|------|---------|
| SQL query tool | Query quarterly metrics for progress review |
| Data exploration tool | Explore trends across the quarter |

## Workflow

### Step 1 -- Review current quarter

Read available weekly summaries from this quarter. Identify:
- What the quarter's goals were (from OKRs or strategy docs)
- What was actually shipped and accomplished
- Which goals were met, partially met, or missed
- Key decisions made and their outcomes

### Step 2 -- Summarise themes

Across the quarter's work, identify the major themes:
- What topics dominated?
- What customer patterns emerged (from churn reviews, Slack)?
- What competitive shifts happened (from competitor reports)?
- What surprised us?

### Step 3 -- Assess metrics trajectory

Query quarterly metrics and compare against goals:
- Where did we improve?
- Where did we stall or decline?
- What's the trajectory heading into next quarter?

### Step 4 -- Draft next quarter priorities

Based on the review, suggest:
- 3-5 themes or priorities for next quarter
- For each: why it matters, what success looks like, and what it connects to
- Carry-forward items that didn't get done this quarter
- New opportunities identified through the quarter

### Step 5 -- Produce the planning document

## Output

Generate an HTML file using the routine template at `skills/system/routine-html-template.md`. Read that file for the full CSS and component patterns.

**File:** Save to `your output folder/quarter-plan-Q[N]-YYYY.html`
**Also save** a markdown copy to `your output folder/q[N]-[YYYY]-planning.md` for archival.
**Open:** Run `open` on the HTML file so it launches in the browser.
**Inline:** Also print a 4-5 line summary in the conversation.

### HTML structure for this skill

```html
<div class="header">
  <h1>Quarterly Planning</h1>
  <div class="subtitle">Q[N] [YYYY] Review + Q[N+1] Priorities</div>
</div>

<div class="section">
  <div class="section-title">Quarter in Review</div>
  <div class="summary-text">[Brief summary of goals and whether they were met]</div>
</div>

<div class="section">
  <div class="section-title">What We Shipped</div>
  <div class="card success">
    <div class="card-title">[Item]</div>
    <div class="card-body">[Impact if known]</div>
  </div>
</div>

<div class="section">
  <div class="section-title">What We Didn't Get To</div>
  <div class="card warning">
    <div class="card-title">[Item]</div>
    <div class="card-body">[Why, and whether it still matters]</div>
  </div>
</div>

<div class="section">
  <div class="section-title">Metrics Trajectory</div>
  <table>
    <thead>
      <tr><th>Metric</th><th>Start of Q</th><th>End of Q</th><th>Change</th><th>Target</th><th>Hit?</th></tr>
    </thead>
    <tbody>
      <tr>
        <td>[Metric]</td><td>[Value]</td><td>[Value]</td>
        <td><span class="badge [green/red]">[+/- %]</span></td>
        <td>[Target]</td>
        <td><span class="badge [green/red]">[Yes/No]</span></td>
      </tr>
    </tbody>
  </table>
</div>

<div class="section">
  <div class="section-title">Key Themes</div>
  <ol class="priority-list">
    <li>
      <div class="card-title">[Theme]</div>
      <div class="card-body">[What we learned]</div>
    </li>
  </ol>
</div>

<div class="section">
  <div class="section-title">Competitive Landscape</div>
  <div class="summary-text">[2-3 sentences on how the picture changed]</div>
</div>

<hr class="divider">

<div class="section">
  <div class="section-title">Suggested Priorities for Q[N+1]</div>
  <ol class="priority-list">
    <li>
      <div class="card-title">[Priority]</div>
      <div class="card-body">[Why it matters. What success looks like.]</div>
    </li>
  </ol>
</div>

<div class="section">
  <div class="section-title">Open Questions</div>
  <ul class="item-list">
    <li>[Question that needs discussion before finalising]</li>
  </ul>
</div>
```

## Guidelines

- This is a draft, not a final plan. Present it as "here's a starting point for your thinking"
- Be honest about what didn't work. Don't spin misses
- Connect priorities to evidence (metrics, customer feedback, competitive moves)
- If data for the quarter is incomplete, say so and work with what's available
- Create the `your output folder/` directory if it doesn't exist

## Change Log

| Date | Change |
|------|--------|
| 13 Mar 2026 | Initial version |
