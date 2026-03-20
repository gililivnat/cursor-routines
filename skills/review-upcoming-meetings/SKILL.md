---
name: review-upcoming-meetings
description: Reviews today's upcoming meetings, prepares context and talking points for each, highlights important and customer-facing meetings, and offers to run customer interview prep. Used in start-of-day routine (step 7).
---

# Review Upcoming Meetings

## When to Use

This skill is called by:
- **Start-of-day routine** (step 7): scans today's calendar, highlights important meetings, and offers customer prep

It can also be run independently when asked to check the calendar or prepare for meetings.

## Inputs

- **Date:** today's date
- **Previous meeting notes** from `your output folder/` (for recurring meetings)
- **Product context** from `your product strategy documentation`

## Tools

### Google Workspace MCP

| Tool | Purpose |
|------|---------|
| Calendar list events tool | List today's calendar events with attendees and details |
| Calendar get event tool | Get full details for a specific event (if needed) |

## Workflow

### Step 1 -- Get today's meetings

List all events for today from the primary calendar:

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

Replace `YYYY-MM-DD` with today's date. Adjust timezone offset if needed based on your timezone (GMT/BST).

### Step 2 -- Categorise meetings

For each meeting, determine:

| Property | How to assess |
|----------|---------------|
| **Type** | 1:1, team sync, cross-team, customer/external, all-hands |
| **Recurring or one-off** | Check if the event has recurrence data |
| **Your role** | Organiser = likely presenter; few attendees = active contributor; large meeting = listener |
| **Customer meeting** | Any attendee with a non-internal email domain (excluding known partners, recruiters, tools like Zoom/Google) |

### Step 3 -- Identify customer meetings

A meeting is a **customer meeting** if:
- At least one attendee has an email domain that is NOT your company domain, `google.com`, `zoom.us`, `calendly.com`, `microsoft.com`, or other common tool/service domains
- The meeting title or description contains keywords like "interview", "customer", "demo", "onboarding", "call with", "sync with [external company]"
- It is a 1:1 or small-group meeting (not a large internal all-hands or company event)

For each identified customer meeting:
1. Extract the external attendee's email domain
2. Note the company name (from the domain or meeting title)
3. Flag it prominently in the output
4. **Offer to run the `prepare-customer-interview` skill** (`skills/prepare-customer-interview/SKILL.md`) for that customer

### Step 4 -- Prepare context for important meetings

For each non-trivial meeting (skip recurring standups with no agenda), write 3-4 lines covering:
- What the meeting is about (from event description, title, or past notes)
- What you should prepare or bring
- Key topics or decisions expected
- Any relevant context from recent Slack discussions or data

For recurring meetings, check `your output folder/` for previous notes and flag continuity items.

### Step 5 -- Identify quiet blocks

Find time ranges with no meetings. These are focus blocks for deep work.

## Output

### When running as part of start-of-day (step 7)

Append a Calendar section to the morning briefing HTML (the file created by step 3, `summarise-slack-and-suggest-tasks`). The morning briefing file is at `your output folder/morning-briefing-YYYY-MM-DD.html`.

Insert this section before the closing `</div>` of the main container, after the existing content:

```html
<hr class="divider">

<div class="section">
  <div class="section-title">Today's Calendar</div>
  <div class="subtitle" style="margin-bottom: 12px">[N] meetings · [N] hours booked · [quiet block summary]</div>

  <!-- Customer meeting (use urgent card + customer prep offer) -->
  <div class="card urgent">
    <div class="card-label urgent">Customer meeting</div>
    <div class="card-title">[Time] -- [Meeting title]</div>
    <div class="card-body">With <span class="person">[External attendee name]</span> ([Company]) · [What to prepare]</div>
    <div class="card-meta">
      <button class="btn-todoist" onclick="this.classList.toggle('added'); this.innerHTML = this.classList.contains('added') ? '<span class=\'icon\'>✓</span> Prep requested' : '<span class=\'icon\'>📋</span> Run customer prep'">
        <span class="icon">📋</span> Run customer prep
      </button>
    </div>
  </div>

  <!-- Important internal meeting -->
  <div class="card highlight">
    <div class="card-label highlight">[Meeting type]</div>
    <div class="card-title">[Time] -- [Meeting title]</div>
    <div class="card-body">[Context: what it's about, what to prepare]</div>
    <div class="card-meta">[Attendees summary]</div>
  </div>

  <!-- Regular meeting (compact) -->
  <div class="card">
    <div class="card-title">[Time] -- [Meeting title]</div>
    <div class="card-body">[Brief context]</div>
  </div>

  <!-- Quiet blocks -->
  <div class="card success">
    <div class="card-label success">Focus time</div>
    <div class="card-title">[Time range]</div>
    <div class="card-body">[Duration] of uninterrupted time</div>
  </div>

  <!-- If no meetings -->
  <p class="quiet">No meetings today. Full day of focus time.</p>
</div>
```

Use `urgent` cards for customer meetings, `highlight` for important internal meetings, and plain cards for routine meetings. Show quiet blocks as `success` cards.

**Customer prep offer:** After generating the HTML, if any customer meetings were found, also print a message in the conversation:

```
Found [N] customer meeting(s) today:
- [Time] -- [Meeting title] with [Company]

Would you like me to run a customer interview prep for any of these?
```

Wait for the user's response. If he confirms, read and execute `skills/prepare-customer-interview/SKILL.md` for the specified customer.

### When running standalone

Generate a standalone HTML file using the routine template at `skills/system/routine-html-template.md`.

**File:** Save to `your output folder/calendar-review-YYYY-MM-DD.html`
**Open:** Run `open` on the file so it launches in the browser.

### Inline summary

Always print a 2-3 line summary in the conversation, regardless of output mode:

```
Today: [N] meetings ([times range]). [Quiet block info].
[Notable meeting highlights].
[Customer meeting flag if applicable].
```

## Guidelines

- Focus prep on meetings where the user needs to actively contribute
- For recurring meetings, check previous notes for continuity
- Flag meetings that look like they could be skipped or delegated
- Customer meetings always get top priority in the output
- Do not include declined events
- When offering customer prep, be specific about which customer and what data would be pulled
## Change Log

| Date | Change |
|------|--------|
| 20 Mar 2026 | Activated. Full rewrite with Google Workspace MCP calendar tools, customer meeting detection, customer interview prep offer, HTML output |
| 13 Mar 2026 | Initial version (placeholder). Awaiting Calendar MCP connection. |
