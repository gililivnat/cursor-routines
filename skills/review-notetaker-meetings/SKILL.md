---
name: review-notetaker-meetings
description: Guide for using the AI notetaker via MCP. Covers what the tool can do, how to call it, and how to synthesise meeting data into actionable outputs.
---

# Using the Monday AI Notetaker

## What It Is

The AI notetaker records meetings, generates transcripts, and extracts summaries, topics, and action items automatically. Access it via the your meeting retrieval tool tool on the your meeting notetaker MCP server.

## What You Can Do

| Capability | How |
|------------|-----|
| List recent meetings | Call with default params |
| Search for a meeting | Use `search` (matches title, participant name, or email) |
| Fetch specific meetings | Use `ids` with one or more meeting IDs |
| Get AI summaries | Set `include_summary: true` |
| Get discussion topics | Set `include_topics: true` |
| Get action items | Set `include_action_items: true` |
| Get full transcript | Set `include_transcript: true` (large payload, use sparingly) |
| Filter by access level | Set `access` to `OWN`, `SHARED_WITH_ME`, `SHARED_WITH_ACCOUNT`, or `ALL` |
| Paginate results | Use `limit` (1-100, default 25) and `cursor` for next pages |

## How to Call It

Use `CallMcpTool` with server your meeting notetaker MCP and tool your meeting retrieval tool.

**Minimal call (list own meetings):**

```json
{
  "server": "your-notetaker-mcp",
  "toolName": "get_notetaker_meetings",
  "arguments": {}
}
```

**Typical call (recent meetings with summaries and action items):**

```json
{
  "server": "your-notetaker-mcp",
  "toolName": "get_notetaker_meetings",
  "arguments": {
    "access": "ALL",
    "limit": 10,
    "include_summary": true,
    "include_action_items": true,
    "include_topics": true
  }
}
```

**Search for a specific meeting:**

```json
{
  "server": "your-notetaker-mcp",
  "toolName": "get_notetaker_meetings",
  "arguments": {
    "search": "sprint planning",
    "include_summary": true,
    "include_action_items": true
  }
}
```

**Fetch by ID (for follow-ups or deep-dives):**

```json
{
  "server": "your-notetaker-mcp",
  "toolName": "get_notetaker_meetings",
  "arguments": {
    "ids": ["meeting-id-here"],
    "include_summary": true,
    "include_topics": true,
    "include_action_items": true,
    "include_transcript": true
  }
}
```

## Parameter Reference

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `ids` | string[] | — | Fetch specific meetings by ID |
| `access` | enum | `OWN` | `OWN`, `SHARED_WITH_ME`, `SHARED_WITH_ACCOUNT`, `ALL` |
| `limit` | number | 25 | Results per page (1-100) |
| `cursor` | string | — | Pagination cursor from previous response |
| `search` | string | — | Search by title, participant name, or email |
| `include_summary` | boolean | false | Include AI-generated summary |
| `include_topics` | boolean | false | Include discussion topics |
| `include_action_items` | boolean | false | Include extracted action items |
| `include_transcript` | boolean | false | Include full transcript (large) |

## Synthesising Output

When presenting meeting data to the user:

**For multiple meetings**, extract and consolidate:
- Key decisions across meetings
- Action items grouped by owner
- Recurring themes or topics
- Unresolved items needing follow-up

**For a single meeting**, present:
- Participants, date, and title
- Summary
- Topics discussed with key points
- Action items with owners
- Open questions

Always draft Slack messages first (never send directly). Save substantial digests to `your output folder/meeting-digest-YYYY-MM-DD.md`.

## Output Formats

Generate HTML content using the routine template at `skills/system/routine-html-template.md`. Read that file for the full CSS and component patterns.

**If running as part of end-of-day routine:** append to the EOD summary HTML file.
**If running standalone:** save to `your output folder/meeting-digest-YYYY-MM-DD.html` and open it.

### Multi-meeting digest HTML

```html
<div class="section">
  <div class="section-title">Meeting Digest · [N] meetings · [date range]</div>

  <div class="card highlight">
    <div class="card-label highlight">[Meeting title] · [date]</div>
    <div class="card-body">[1-2 sentence summary]</div>
    <div class="card-meta">Decisions: [if any]</div>
  </div>
  <!-- repeat per meeting -->
</div>

<div class="section">
  <div class="section-title">Action Items</div>
  <div class="task-row">
    <div class="task-content"><span class="person">[Owner]</span>: [action] <span class="quiet">(from: [meeting])</span></div>
    <button class="btn-todoist" onclick="this.classList.toggle('added'); this.innerHTML = this.classList.contains('added') ? '<span class=\'icon\'>✓</span> Added' : '<span class=\'icon\'>+</span> Add to your task manager'">
      <span class="icon">+</span> Add to your task manager
    </button>
  </div>
</div>

<div class="section">
  <div class="section-title">Themes Across Meetings</div>
  <ul class="item-list">
    <li>[Pattern or recurring topic]</li>
  </ul>
</div>
```

### Single meeting HTML

```html
<div class="section">
  <div class="section-title">[Meeting title] · [date]</div>
  <p class="quiet">Participants: [list]</p>

  <div class="card highlight">
    <div class="card-title">Summary</div>
    <div class="card-body">[AI summary]</div>
  </div>

  <div class="section-title" style="margin-top:20px">Topics</div>
  <ol class="priority-list">
    <li>
      <div class="card-title">[Topic]</div>
      <div class="card-body">[Key points]</div>
    </li>
  </ol>

  <div class="section-title" style="margin-top:20px">Action Items</div>
  <div class="task-row">
    <div class="task-content"><span class="person">[Owner]</span>: [action]</div>
    <button class="btn-todoist" onclick="this.classList.toggle('added'); this.innerHTML = this.classList.contains('added') ? '<span class=\'icon\'>✓</span> Added' : '<span class=\'icon\'>+</span> Add to your task manager'">
      <span class="icon">+</span> Add to your task manager
    </button>
  </div>

  <div class="section-title" style="margin-top:20px">Open Questions</div>
  <ul class="item-list">
    <li>[Unresolved item]</li>
  </ul>
</div>
```

## Checking for Tool Updates

The notetaker MCP tool may gain new parameters or capabilities over time. To check for changes:

1. Read the tool descriptor at:
   `[your-workspace-path] user-Workspace/mcps/your-notetaker-mcp/tools/get_notetaker_meetings.json`

2. Compare the descriptor's `properties` against the Parameter Reference table above.

3. If new parameters exist or descriptions have changed:
   - Update the Parameter Reference table in this file
   - Update the "What You Can Do" table if new capabilities were added
   - Update the example calls if relevant
   - Note what changed at the bottom of this file under Change Log

4. If parameters have been removed or renamed, update all references accordingly.

**When to check:** At the start of any session where this skill is used. A quick read of the descriptor file is cheap and ensures the skill stays current.

## Change Log

| Date | Change |
|------|--------|
| 12 Mar 2026 | Initial version. Documented all parameters from MCP tool descriptor |

## Guidelines

- Write in British English
- Never use em dashes
- Be concise: summaries should be scannable
- Only include transcript content if the user specifically asks (transcripts are large)
- Flag if any meetings couldn't be retrieved or if data seems incomplete
any written output
