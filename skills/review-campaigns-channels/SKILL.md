---
name: review-campaigns-channels
description: Finds and reads all Slack channels with "Product" in the name. Summarises recent activity, flags important threads. Use as part of the daily start-of-day routine.
---

# Review Product Channels

## When to Use

This skill is called by the start-of-day routine (step 2). It can also be run independently when asked to review product Slack activity.

## Inputs

- Search term: "CHANNEL_KEYWORD"
- Time window: last 24 hours from now
- Must-check channels (always include even if the search misses them):
  - `ask-product`
  - `product-churns`

## Tools

### Slack MCP

| Tool | Purpose |
|------|---------|
| `slack_search_channels` | Find channels with "CHANNEL_KEYWORD" in the name |
| `slack_read_channel` | Read recent messages from each channel |
| `slack_read_thread` | Read thread replies for important conversations |

## Workflow

### Step 1 -- Find channels

```
Tool: slack_search_channels
Arguments:
  query: "CHANNEL_KEYWORD"
  channel_types: "public_channel,private_channel"
  include_archived: false
```

Note all channel IDs and names returned.

**Must-check fallback:** If any of the must-check channels (`ask-product`, `product-churns`) are missing from the search results, search for them individually:

```
Tool: slack_search_channels
Arguments:
  query: "[channel-name]"
  channel_types: "public_channel,private_channel"
  include_archived: false
```

Merge the results. If a must-check channel still cannot be found, flag it explicitly in the output as missing rather than silently skipping it.

### Step 2 -- Read recent messages

For each channel found, read messages from the last 24 hours:

```
Tool: slack_read_channel
Arguments:
  channel_id: [channel ID]
  oldest: [Unix timestamp for 24 hours ago]
  limit: 100
```

If a channel has more than 100 messages, paginate using the `cursor` parameter.

### Step 3 -- Read important threads

For messages with thread replies (especially those with multiple responses or reactions), read the full thread:

```
Tool: slack_read_thread
Arguments:
  channel_id: [channel ID]
  message_ts: [thread timestamp]
```

### Step 3b -- Check threads the user is part of

For each product channel, search for threads the user has participated in with recent activity:

```
Tool: slack_search_public
Arguments:
  query: "from:<@YOUR_SLACK_USER_ID> is:thread in:<#[channel_id]>"
  sort: "timestamp"
  sort_dir: "desc"
  after: [Unix timestamp for 24 hours ago]
```

For each thread found, read it with `slack_read_thread` and check whether there are new replies after the user's last message. Flag any that look like they need a quick response.

### Step 4 -- Analyse and summarise

For each channel, identify:
- **Key discussions:** What topics were actively discussed?
- **Decisions made:** Were any decisions reached?
- **Questions asked:** Were there unanswered questions the user should know about?
- **Threads awaiting the user's reply:** Threads the user participated in where someone has replied since and may be waiting for a response
- **Escalations:** Anything flagged as urgent or blocked?
- **Notable activity:** Unusual volume, new topics, or cross-team threads

### Step 5 -- Produce summary

Write a summary grouped by channel.

## Output

Generate HTML content using the routine HTML template at `skills/system/routine-html-template.md`. Read that file for the full CSS and component patterns.

**Note:** If running as part of the start-of-day routine, hold the HTML content and let the `summarise-slack-and-suggest-tasks` skill combine all sections into one morning briefing file.

### HTML structure for this skill

```html
<div class="section">
  <div class="section-title">Product Channels · Last 24 hours · [N] channels checked</div>

  <!-- One card per active channel -->
  <div class="card highlight">
    <div class="card-label highlight">[#channel-name] · [N] messages</div>
    <div class="card-title">[Key topic or thread]</div>
    <div class="card-body">[1-2 sentence summary]</div>
    <div class="card-meta">Action needed: [if any, otherwise omit this line]</div>
  </div>

  <!-- For channels with urgent/blocked items -->
  <div class="card urgent">
    <div class="card-label urgent">[#channel-name] · Action needed</div>
    <div class="card-title">[Topic requiring your input]</div>
    <div class="card-body">[What's needed and from whom]</div>
  </div>

  <!-- Threads the user is part of with replies awaiting his response -->
  <div class="card urgent">
    <div class="card-label urgent">Thread reply · needs response</div>
    <div class="card-title"><span class="channel">#[channel]</span> · [Thread topic]</div>
    <div class="card-body">[Who replied and what they said/asked]</div>
    <div class="card-meta">[time ago]</div>
  </div>

  <!-- Quiet channels grouped at the end -->
  <p class="quiet">Quiet: [#channel-1], [#channel-2], [#channel-3]</p>
</div>
```

If no product channels are found, flag this as unexpected and move on.

## Guidelines

- One to two lines per topic, not full recaps
- Lead with the most active or important channels
- Flag anything that looks like it needs direct input
- Note the channel name and message count so the user can gauge activity levels
- Skip bot-only messages unless they contain actionable content

## Change Log

| Date | Change |
|------|--------|
| 16 Mar 2026 | Added must-check channels list (ask-product, product-churns) with fallback search. Channels missing from results are now flagged explicitly |
| 16 Mar 2026 | Added Step 3b: check threads the user is part of for replies needing response |
| 13 Mar 2026 | Initial version |
