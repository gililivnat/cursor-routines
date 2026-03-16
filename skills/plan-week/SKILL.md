---
name: plan-week
description: Reviews priorities, outstanding tasks, upcoming commitments, and last week's context to help set his weekly focus. Use as part of the weekly Monday routine.
---

# Plan Week

## When to Use

This skill is called by the weekly routine on **Monday**. It can also be run independently when asked to plan the week.

## Inputs

- Last week's summary from `your output folder/` (for carry-overs)
- Scheduled tasks due this week from `your scheduled tasks file`
- Product strategy context from `your product strategy documentation`

## Tools

### Slack MCP

| Tool | Purpose |
|------|---------|
| `slack_search_public` | Check for any weekend messages or Monday-morning threads |

### Meeting Notetaker MCP

| Tool | Purpose |
|------|---------|
| Meeting retrieval tool | Check what meetings are coming up this week (if data available) |

## Workflow

### Step 1 -- Review last week

Read the most recent weekly summary from `your output folder/`. Note:
- Open items carried forward
- Trends worth watching
- Decisions that need follow-up

### Step 2 -- Check scheduled tasks

Read `your scheduled tasks file`. Identify:
- Tasks due this week (one-off with approaching due dates)
- Recurring tasks that run this week (Monday tasks, Friday tasks)
- Overdue tasks that haven't been completed

### Step 3 -- Scan for Monday context

Check Slack for any weekend activity or early Monday threads that set the tone for the week.

### Step 4 -- Draft the weekly plan

Produce a short, prioritised focus list.

## Output

Generate an HTML file using the routine template at `skills/system/routine-html-template.md`. Read that file for the full CSS and component patterns.

**File:** Save to `your output folder/week-plan-YYYY-MM-DD.html`
**Open:** Run `open` on the file so it launches in the browser.
**Inline:** Also print a 2-3 line summary in the conversation.

### HTML structure for this skill

```html
<div class="header">
  <h1>Week Plan</h1>
  <div class="subtitle">Week of [date] · [N] carry-overs · [N] scheduled tasks</div>
</div>

<div class="section">
  <div class="section-title">Top Priorities</div>
  <ol class="priority-list">
    <li>
      <div class="card-title">[Highest priority]</div>
      <div class="card-body">[Why, and what "done" looks like]</div>
    </li>
    <li>
      <div class="card-title">[Second priority]</div>
      <div class="card-body">[Why, and what "done" looks like]</div>
    </li>
    <li>
      <div class="card-title">[Third priority]</div>
      <div class="card-body">[Why, and what "done" looks like]</div>
    </li>
  </ol>
</div>

<hr class="divider">

<div class="section">
  <div class="section-title">Carry-overs from Last Week</div>
  <div class="task-row">
    <div class="task-content">[Item] <span class="badge amber">[Status]</span></div>
    <button class="btn-todoist" onclick="this.classList.toggle('added'); this.innerHTML = this.classList.contains('added') ? '<span class=\'icon\'>✓</span> Added' : '<span class=\'icon\'>+</span> Add to your task manager'">
      <span class="icon">+</span> Add to your task manager
    </button>
  </div>
</div>

<div class="section">
  <div class="section-title">Scheduled Tasks This Week</div>
  <table>
    <thead>
      <tr><th>Task</th><th>Owner</th><th>Due</th></tr>
    </thead>
    <tbody>
      <tr><td>[Task]</td><td><span class="badge cyan">[Agent/User]</span></td><td>[Day]</td></tr>
    </tbody>
  </table>
</div>

<div class="section">
  <div class="section-title">Key Meetings</div>
  <ul class="item-list">
    <li><strong>[Meeting]</strong> · [Day, time if known]</li>
  </ul>
</div>

<div class="section">
  <div class="section-title">One Thing to Watch</div>
  <div class="card warning">
    <div class="card-title">[Emerging trend, risk, or opportunity]</div>
    <div class="card-body">[From last week's data]</div>
  </div>
</div>
```

## Guidelines

- Keep it to 3 top priorities. More than that is no longer prioritised
- "Done" should be concrete, not vague ("Ship the email builder fix" not "Work on email builder")
- If last week's summary isn't available, work with what you have and note the gap
- This is a planning aid, not a command. The user decides the final priorities
## Change Log

| Date | Change |
|------|--------|
| 13 Mar 2026 | Initial version |
