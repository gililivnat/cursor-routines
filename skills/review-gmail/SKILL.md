---
name: review-gmail
description: Scans Gmail inbox for urgent or important emails and summarises what needs attention. PLACEHOLDER -- requires a Gmail MCP connection to activate.
---

# Review Gmail

> **Status: INACTIVE.** This skill requires a Gmail MCP connection that is not yet configured. The skill definition is ready; once the MCP is connected, uncomment `review_gmail` in `routines/state.md` to activate.

## When to Use

This skill is called by the start-of-day routine (step 6) once activated. It scans the user's inbox for anything urgent or important.

## Inputs

- Gmail MCP (not yet connected)
- Time window: last 24 hours

## Tools

### Gmail MCP (to be configured)

| Tool | Purpose |
|------|---------|
| TBD | List recent emails / search inbox |
| TBD | Read email content |
| TBD | Get email metadata (sender, subject, labels) |

## Workflow

### Step 1 -- Fetch recent emails

Query the Gmail MCP for emails received in the last 24 hours, ordered by most recent first.

### Step 2 -- Filter and prioritise

Categorise emails by urgency:
- **Urgent:** from known stakeholders, contains action words ("please review", "deadline", "ASAP"), flagged/starred
- **Important:** from team members, contains product-related keywords, meeting invites with agendas
- **FYI:** newsletters, automated notifications, CC'd threads

### Step 3 -- Summarise urgent and important

For urgent and important emails, extract:
- Sender and subject
- What they're asking for (if anything)
- Whether a reply is needed and by when

### Step 4 -- Produce summary

```
**Gmail** | Last 24 hours | [N] emails

**Needs response:**
- [Sender]: [Subject] -- [What they need]
- ...

**Worth reading:**
- [Sender]: [Subject] -- [1-line summary]
- ...

**Skippable:** [N] newsletters/notifications
```

## Guidelines

- Do not read the full body of every email. Subject + sender + first few lines is enough for triage
- Flag anything from leadership or external partners as high priority
- Do not summarise automated notifications unless they contain actionable content

## Activation Checklist

When ready to activate:
1. Connect a Gmail MCP in Cursor Settings
2. Update the tool table above with actual tool names and parameters
3. Uncomment `review_gmail: false` in `routines/state.md`
4. Test with a single run before relying on it daily

## Change Log

| Date | Change |
|------|--------|
| 13 Mar 2026 | Initial version (placeholder). Awaiting Gmail MCP connection. |
