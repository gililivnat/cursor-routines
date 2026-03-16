---
name: summarise-campaigns-day
description: Reviews product-related Slack messages from today, produces a summary with highlights, key decisions, and follow-up items. Use as part of the daily end-of-day routine.
---

# Summarise Product Day

## When to Use

This skill is called by the end-of-day routine (step 1). It can also be run independently when asked to summarise today's product activity.

## Inputs

- Time window: today (from midnight or start of working day)
- Channels: all Slack channels with "CHANNEL_KEYWORD" in the name

## Tools

### Slack MCP

| Tool | Purpose |
|------|---------|
| `slack_search_channels` | Find channels with "CHANNEL_KEYWORD" in the name |
| `slack_read_channel` | Read today's messages from each channel |
| `slack_read_thread` | Read thread replies for key discussions |
| `slack_search_public` | Search for specific topics or mentions across channels |

## Workflow

### Step 1 -- Find product channels

```
Tool: slack_search_channels
Arguments:
  query: "CHANNEL_KEYWORD"
  channel_types: "public_channel,private_channel"
  include_archived: false
```

### Step 2 -- Read today's messages

For each channel, read messages from today:

```
Tool: slack_read_channel
Arguments:
  channel_id: [channel ID]
  oldest: [Unix timestamp for start of today]
  limit: 100
```

Paginate if needed.

### Step 3 -- Deep-dive on important threads

For threads with multiple replies, decisions, or escalations, read the full thread.

### Step 3b -- Check threads the user is part of

Search for threads the user participated in within product channels today:

```
Tool: slack_search_public
Arguments:
  query: "from:<@YOUR_SLACK_USER_ID> is:thread in:<#[channel_id]>"
  sort: "timestamp"
  sort_dir: "desc"
  after: [Unix timestamp for start of today]
```

For each thread found, read it with `slack_read_thread`. Check for new replies after the user's last message that may still need a response before end of day.

### Step 4 -- Analyse the day

Across all channels, identify:
- **Decisions made:** What was decided today?
- **Progress updates:** What moved forward?
- **Issues raised:** New bugs, blockers, or concerns?
- **Customer mentions:** Any customer feedback, wins, or complaints surfaced?
- **Cross-team coordination:** Threads involving other teams (CRM, WM, platform)?
- **Threads still awaiting the user's reply:** Threads the user is part of where someone has replied and may be waiting for a response
- **Unresolved items:** Questions or discussions still open at end of day

### Step 5 -- Produce summary

Write a concise end-of-day summary.

## Output

Generate an HTML file using the routine template at `skills/system/routine-html-template.md`. Read that file for the full CSS and component patterns.

**File:** Save to `your output folder/eod-summary-YYYY-MM-DD.html`
**Open:** Run `open` on the file so it launches in the browser.
**Inline:** Also print a 2-3 line summary in the conversation.

### HTML structure for this skill

```html
<div class="header">
  <h1>End of Day</h1>
  <div class="subtitle">[Day, Date] · [N] channels reviewed</div>
</div>

<div class="section">
  <div class="section-title">Key Decisions</div>
  <div class="card success">
    <div class="card-title">[Decision]</div>
    <div class="card-body">[Context]</div>
    <div class="card-meta"><span class="channel">#[channel]</span></div>
  </div>
</div>

<div class="section">
  <div class="section-title">Progress</div>
  <div class="card highlight">
    <div class="card-title">[What moved forward]</div>
    <div class="card-body">[Brief detail]</div>
  </div>
</div>

<div class="section">
  <div class="section-title">Issues to Watch</div>
  <div class="card warning">
    <div class="card-title">[Issue or blocker]</div>
    <div class="card-body">[Status and who's on it]</div>
  </div>
</div>

<div class="section">
  <div class="section-title">Threads Still Awaiting Reply</div>
  <!-- Only shown if there are threads with unanswered replies -->
  <div class="card warning">
    <div class="card-label warning">Unanswered</div>
    <div class="card-title"><span class="channel">#[channel]</span> · [Thread topic]</div>
    <div class="card-body">[Who is waiting and what they asked]</div>
    <div class="card-meta">[time ago]</div>
  </div>
</div>

<div class="section">
  <div class="section-title">Carry to Tomorrow</div>
  <div class="task-row">
    <div class="task-content">[Unresolved question or pending action]</div>
    <button class="btn-todoist" onclick="this.classList.toggle('added'); this.innerHTML = this.classList.contains('added') ? '<span class=\'icon\'>✓</span> Added' : '<span class=\'icon\'>+</span> Add to your task manager'">
      <span class="icon">+</span> Add to your task manager
    </button>
  </div>
</div>
```

If it was a quiet day: `<p class="quiet">Quiet day across product channels. Nothing to flag.</p>`

## Guidelines

- This is a retrospective, not a live report. Summarise, don't replay
- Focus on what changed or what matters, not a chronological log
- If a topic from the morning start-of-day review progressed during the day, note the update
- Keep it under 200 words unless something significant happened
## Change Log

| Date | Change |
|------|--------|
| 16 Mar 2026 | Added Step 3b: check threads the user is part of for unanswered replies; added "Threads Still Awaiting Reply" EOD section |
| 13 Mar 2026 | Initial version |
