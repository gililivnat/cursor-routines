---
name: review-slack-dms
description: Reviews personal Slack direct messages from the last 24 hours. Flags important messages and summarises key conversations. Use as part of the daily start-of-day routine.
---

# Review Slack DMs

## When to Use

This skill is called by the start-of-day routine (step 1). It can also be run independently when asked to check Slack DMs.

## Inputs

- Your Slack user ID: set `YOUR_SLACK_USER_ID` below
- Time window: last 24 hours from now

## Tools

### Slack MCP

| Tool | Purpose |
|------|---------|
| `slack_read_channel` | Read DM history by passing a user_id as `channel_id` |
| `slack_search_public` | Search for messages sent to the user (`to:<@YOUR_SLACK_USER_ID>`) |
| `slack_read_thread` | Read thread replies for deeper context on important messages |
| `slack_search_users` | Look up user details when needed |

## Workflow

### Step 1 -- Find recent DMs

Search for messages directed to the user in the last 24 hours:

```
Tool: slack_search_public
Arguments:
  query: "to:<@YOUR_SLACK_USER_ID>"
  sort: "timestamp"
  sort_dir: "desc"
  after: [Unix timestamp for 24 hours ago]
```

Also search for messages from the user to capture full conversations:

```
Tool: slack_search_public
Arguments:
  query: "from:<@YOUR_SLACK_USER_ID>"
  sort: "timestamp"
  sort_dir: "desc"
  after: [Unix timestamp for 24 hours ago]
```

### Step 2 -- Find threads the user is part of

Search for threads the user has participated in across all channels in the last 24 hours. This catches threads where someone may have replied and is waiting for a response.

```
Tool: slack_search_public
Arguments:
  query: "from:<@YOUR_SLACK_USER_ID> is:thread"
  sort: "timestamp"
  sort_dir: "desc"
  after: [Unix timestamp for 24 hours ago]
```

For each thread found, read the full thread using `slack_read_thread`. Check whether there are replies **after** the user's last message in the thread — these are the ones that may need a quick response.

### Step 3 -- Read important threads

For messages from Step 1 with thread replies, read the full thread for context:

```
Tool: slack_read_thread
Arguments:
  channel_id: [channel from message]
  message_ts: [thread timestamp]
```

### Step 4 -- Categorise and prioritise

For each conversation (DMs and thread replies), assess:
- **Urgency:** Does this need a reply today?
- **Topic:** What is this about? (product, team, ops, social)
- **Action needed:** Is the user expected to do something?
- **Thread waiting:** Is someone waiting for the user's reply in a thread?

### Step 5 -- Produce summary

Write a concise summary grouped by urgency.

## Output

Generate an HTML file using the routine HTML template at `skills/system/routine-html-template.md`. Read that file for the full CSS and component patterns.

**File:** Save to `your output folder/morning-briefing-YYYY-MM-DD.html`
**Open:** Run `open` on the file so it launches in the browser.
**Inline:** Also provide a 2-3 line plain-text summary in the conversation.

**Note:** This skill's HTML output may be combined with other morning routine outputs (product channels, suggested tasks, metrics) into a single morning briefing HTML file. If running as part of the start-of-day routine, hold the HTML content and let the `summarise-slack-and-suggest-tasks` skill combine all sections into one file.

### HTML structure for this skill

```html
<div class="section">
  <div class="section-title">Slack DMs · Last 24 hours</div>

  <!-- Repeat for each DM needing a response -->
  <div class="card urgent">
    <div class="card-label urgent">Needs reply</div>
    <div class="card-title"><span class="person">[Person]</span> · [Topic]</div>
    <div class="card-body">[1-line summary of what they need]</div>
    <div class="card-meta">[time ago]</div>
  </div>

  <!-- FYI items use the highlight card -->
  <div class="card highlight">
    <div class="card-label highlight">FYI</div>
    <div class="card-title"><span class="person">[Person]</span> · [Topic]</div>
    <div class="card-body">[1-line summary]</div>
    <div class="card-meta">[time ago]</div>
  </div>

  <!-- Threads to watch use the warning card -->
  <div class="card warning">
    <div class="card-label warning">Watch</div>
    <div class="card-title">[Thread description]</div>
    <div class="card-body">[Why it matters]</div>
  </div>

  <!-- Threads the user is part of with new replies awaiting response -->
  <div class="card urgent">
    <div class="card-label urgent">Thread reply</div>
    <div class="card-title"><span class="channel">#[channel]</span> · [Thread topic]</div>
    <div class="card-body">[Who replied and what they said/asked]</div>
    <div class="card-meta">[time ago]</div>
  </div>
</div>
```

If there are no messages, output: `<p class="quiet">No DMs in the last 24 hours.</p>`

## Guidelines

- Be concise: one line per conversation, not paragraphs
- Prioritise by urgency, not chronology
- If a message looks like it needs your input, flag it clearly
- Do not read messages older than 24 hours unless a thread started within the window

## Change Log

| Date | Change |
|------|--------|
| 16 Mar 2026 | Added Step 2: search for threads the user is part of with new replies needing response |
| 13 Mar 2026 | Initial version |
