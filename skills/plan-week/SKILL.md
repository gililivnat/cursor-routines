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
- This week's calendar events (for meeting-aware planning and customer prep)

## Tools

### Slack MCP

| Tool | Purpose |
|------|---------|
| `slack_search_public` | Check for any weekend messages or Monday-morning threads |

### Google Workspace MCP

| Tool | Purpose |
|------|---------|
| Calendar list events tool | List this week's calendar events with attendees and details |

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

### Step 3 -- Scan the week's calendar

Query the full week's calendar using Google Workspace MCP:

```json
{
  "server": "your-google-workspace-mcp",
  "toolName": "calendar_listEvents",
  "arguments": {
    "calendarId": "primary",
    "timeMin": "YYYY-MM-DDT00:00:00Z",
    "timeMax": "YYYY-MM-DDT23:59:59Z"
  }
}
```

Use this Monday as `timeMin` and Friday as `timeMax` (or Sunday if the user prefers full-week visibility).

From the results:
- Count total meetings and hours booked per day
- Identify the busiest and quietest days
- Flag days with back-to-back meetings (limited focus time)
- **Detect customer meetings:** any meeting where at least one attendee has a non-internal email domain (excluding tool domains like `google.com`, `zoom.us`, `calendly.com`, `microsoft.com`)

For each customer meeting found:
1. Extract the external attendee's company (from domain or meeting title)
2. Note the day and time
3. **Offer to run `prepare-customer-interview`** (`skills/prepare-customer-interview/SKILL.md`) for each customer

### Step 4 -- Scan for Monday context

Check Slack for any weekend activity or early Monday threads that set the tone for the week.

### Step 5 -- Draft the weekly plan

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
  <div class="section-title">This Week's Calendar</div>
  <div class="subtitle" style="margin-bottom: 12px">[N] meetings · [N] hours booked · Busiest day: [day]</div>

  <!-- Customer meetings get urgent cards with prep offer -->
  <div class="card urgent">
    <div class="card-label urgent">Customer meeting</div>
    <div class="card-title">[Day, time] -- [Meeting title]</div>
    <div class="card-body">With <span class="person">[External attendee]</span> ([Company]) · [Brief context]</div>
    <div class="card-meta">
      <button class="btn-todoist" onclick="this.classList.toggle('added'); this.innerHTML = this.classList.contains('added') ? '<span class=\'icon\'>✓</span> Prep requested' : '<span class=\'icon\'>📋</span> Run customer prep'">
        <span class="icon">📋</span> Run customer prep
      </button>
    </div>
  </div>

  <!-- Important internal meetings -->
  <div class="card highlight">
    <div class="card-title">[Day, time] -- [Meeting title]</div>
    <div class="card-body">[Brief context or carry-over topic]</div>
  </div>

  <!-- Day-by-day summary -->
  <table>
    <thead><tr><th>Day</th><th>Meetings</th><th>Hours booked</th><th>Focus time</th></tr></thead>
    <tbody>
      <tr><td>Monday</td><td>[N]</td><td>[N]h</td><td>[N]h</td></tr>
      <tr><td>Tuesday</td><td>[N]</td><td>[N]h</td><td>[N]h</td></tr>
      <tr><td>Wednesday</td><td>[N]</td><td>[N]h</td><td>[N]h</td></tr>
      <tr><td>Thursday</td><td>[N]</td><td>[N]h</td><td>[N]h</td></tr>
      <tr><td>Friday</td><td>[N]</td><td>[N]h</td><td>[N]h</td></tr>
    </tbody>
  </table>
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
| 20 Mar 2026 | Added Google Calendar scan (Step 3): full week calendar view, day-by-day meeting/focus breakdown, customer meeting detection with prep offer |
| 13 Mar 2026 | Initial version |
