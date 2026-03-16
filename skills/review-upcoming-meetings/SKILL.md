---
name: review-upcoming-meetings
description: Reviews today's upcoming meetings and prepares 3-4 lines of context or talking points for each. PLACEHOLDER -- requires a Calendar MCP connection to activate.
---

# Review Upcoming Meetings

> **Status: INACTIVE.** This skill requires a Google Calendar MCP connection that is not yet configured. The skill definition is ready; once the MCP is connected, uncomment `review_upcoming_meetings` in `routines/state.md` to activate.

## When to Use

This skill is called by the start-of-day routine (step 5) once activated. It reviews today's calendar and prepares meeting context.

## Inputs

- Calendar MCP (not yet connected)
- Previous meeting notes from `your output folder/` (for recurring meetings)
- Product context from `your product strategy documentation`

## Tools

### Calendar MCP (to be configured)

| Tool | Purpose |
|------|---------|
| TBD | List today's calendar events |
| TBD | Get event details (attendees, description, links) |

## Workflow

### Step 1 -- Get today's meetings

Query the Calendar MCP for all events today, ordered by start time.

### Step 2 -- Categorise meetings

For each meeting, identify:
- **Type:** 1:1, team sync, cross-team, external, all-hands
- **Recurring or one-off**
- **Your role:** presenter, contributor, listener

### Step 3 -- Prepare context

For each important meeting, write 3-4 lines covering:
- What the meeting is about (from the event description or past notes)
- What you should prepare or bring
- Key topics or decisions expected
- Any relevant context from recent Slack discussions or data

### Step 4 -- Produce summary

```
**Today's Meetings** | [date]

**[Time] -- [Meeting title]** ([attendees])
- [Context line 1]
- [What to prepare]
- [Key topic or decision expected]

**[Time] -- [Meeting title]** ([attendees])
- [Context lines]

**Quiet blocks:** [Time ranges with no meetings]
```

## Guidelines

- Focus prep on meetings where the user needs to actively contribute
- For recurring meetings, check previous notes for continuity
- Flag meetings that look like they could be skipped or delegated

## Activation Checklist

When ready to activate:
1. Connect a Google Calendar MCP in Cursor Settings
2. Update the tool table above with actual tool names and parameters
3. Uncomment `review_upcoming_meetings: false` in `routines/state.md`
4. Test with a single run before relying on it daily

## Change Log

| Date | Change |
|------|--------|
| 13 Mar 2026 | Initial version (placeholder). Awaiting Calendar MCP connection. |
