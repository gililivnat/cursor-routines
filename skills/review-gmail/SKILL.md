---
name: review-gmail
description: Scans Gmail inbox for urgent or important emails and summarises what needs attention, with direct links to each email. Used in start-of-day (step 6) and end-of-day (step 3) routines.
---

# Review Gmail

## When to Use

This skill is called by:
- **Start-of-day routine** (step 6): scans the last 24 hours for anything urgent or important
- **End-of-day routine** (step 3): scans today's emails and highlights what came in during the day

It can also be run independently when asked to check email.

## Inputs

- **Time window:** last 24 hours (morning) or today (EOD)
- **Context:** if running as part of EOD, check what was already flagged in the morning to avoid repeating

## Tools

### Google Workspace MCP

| Tool | Purpose |
|------|---------|
| Email search tool | Search inbox by query (same syntax as Gmail search box) |
| Email get tool | Get full content of a specific email by message ID |

## Email URLs

For every email surfaced, include a direct Gmail link using this format:

```
https://mail.google.com/mail/u/0/#inbox/<messageId>
```

Where `<messageId>` is the ID returned by your email search tool or your email get tool.

## Workflow

### Step 1 -- Fetch recent emails

Search for emails from the relevant time window:

```json
{
  "server": "your-google-workspace-mcp",
  "toolName": "gmail_search",
  "arguments": {
    "query": "newer_than:1d",
    "maxResults": 50,
    "labelIds": ["INBOX"]
  }
}
```

For EOD runs, use `after:YYYY/MM/DD` with today's date instead of `newer_than:1d`.

### Step 2 -- Identify important emails

From the search results, identify emails worth surfacing based on:
- **Urgent:** from known stakeholders (leadership, external partners, direct reports), contains action words ("please review", "deadline", "ASAP", "urgent"), flagged/starred
- **Important:** from team members, contains product-related keywords, meeting invites with agendas, replies to threads the user started
- **FYI:** newsletters, automated notifications, CC'd threads

For urgent and important emails, fetch the full message to get context:

```json
{
  "server": "your-google-workspace-mcp",
  "toolName": "gmail_get",
  "arguments": {
    "messageId": "<id>",
    "format": "full"
  }
}
```

Only fetch full content for emails that look actionable. Do not fetch every email.

### Step 3 -- Produce summary

For each surfaced email, include: sender, subject, what they need, and a clickable link.

## Output

### When running as part of start-of-day (step 6)

Append a Gmail section to the morning briefing HTML (the file created by step 3, `summarise-slack-and-suggest-tasks`). The morning briefing file is at `your output folder/morning-briefing-YYYY-MM-DD.html`.

Insert this section before the closing `</div>` of the main container, after the existing content:

```html
<hr class="divider">

<div class="section">
  <div class="section-title">Gmail</div>
  <div class="subtitle" style="margin-bottom: 12px">[N] emails in last 24 hours · [N] need attention</div>

  <!-- Repeat for each important email -->
  <div class="card warning">
    <div class="card-label warning">Needs response</div>
    <div class="card-title"><a href="https://mail.google.com/mail/u/0/#inbox/[messageId]" target="_blank">[Subject]</a></div>
    <div class="card-body">From [Sender] · [What they need in 1 line]</div>
    <div class="card-meta">[time received]</div>
  </div>

  <div class="card highlight">
    <div class="card-label highlight">Worth reading</div>
    <div class="card-title"><a href="https://mail.google.com/mail/u/0/#inbox/[messageId]" target="_blank">[Subject]</a></div>
    <div class="card-body">From [Sender] · [1-line summary]</div>
    <div class="card-meta">[time received]</div>
  </div>

  <!-- If nothing important -->
  <p class="quiet">Nothing urgent. [N] newsletters/notifications skipped.</p>
</div>
```

Use `warning` cards for emails needing a response, `highlight` cards for worth-reading, and skip FYI/newsletters entirely (just mention the count).

### When running as part of end-of-day (step 3)

Append a Gmail section to the EOD summary HTML file at `your output folder/eod-summary-YYYY-MM-DD.html`. Use the same card structure as above but with a focus on what came in during the day:

```html
<div class="section">
  <div class="section-title">Email Highlights</div>

  <div class="card warning">
    <div class="card-label warning">Still unanswered</div>
    <div class="card-title"><a href="https://mail.google.com/mail/u/0/#inbox/[messageId]" target="_blank">[Subject]</a></div>
    <div class="card-body">From [Sender] · [What they need]</div>
  </div>

  <div class="card success">
    <div class="card-label success">Handled</div>
    <div class="card-title"><a href="https://mail.google.com/mail/u/0/#inbox/[messageId]" target="_blank">[Subject]</a></div>
    <div class="card-body">From [Sender] · [Brief outcome]</div>
  </div>
</div>
```

For EOD, try to distinguish between emails that were dealt with during the day (mark as "Handled") and those still unanswered (mark as "Still unanswered"). If you cannot determine reply status, default to listing them as important.

### When running standalone

Generate a standalone HTML file using the routine template at `skills/system/routine-html-template.md`.

**File:** Save to `your output folder/gmail-review-YYYY-MM-DD.html`
**Open:** Run `open` on the file so it launches in the browser.

### Inline summary

Always print a 2-3 line summary in the conversation, regardless of output mode.

## Guidelines

- Do not read the full body of every email. Subject + sender + first few lines is enough for triage
- Flag anything from leadership or external partners as high priority
- Do not summarise automated notifications unless they contain actionable content
- Every surfaced email must include a clickable link to the Gmail message
## Change Log

| Date | Change |
|------|--------|
| 19 Mar 2026 | Activated. Rewrote with Google Workspace MCP tools, email URL links, SOD/EOD dual-use, HTML output |
| 13 Mar 2026 | Initial version (placeholder). Awaiting Gmail MCP connection. |
