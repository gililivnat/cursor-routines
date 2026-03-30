---
name: prepare-biweekly-update
description: Prepares for the bi-weekly director update meeting by reviewing the latest deck, pulling current metrics, scanning Slack for feature/PR discussions, and drafting a prep message to the team. Owned by the Product Ops agent.
---

# Prepare Bi-weekly Update

## When to Use

This skill is triggered by the `/biweekly` slash command. Run it 1-2 days before the Thursday bi-weekly director update meeting. It can also be run independently when asked to prepare for the bi-weekly.

## Purpose

The bi-weekly update is a meeting for directors (Dev, Product, Design) where each team member presents their section. This skill:

1. Reviews the most recent presentation from the archive
2. Pulls current metrics and compares them with the last meeting's numbers
3. Scans Slack for features shipped, PRs discussed, and research findings
4. Drafts a Slack message to the team asking each person to update their slides, with context on what to include

## Inputs

- Data guide: `your data guide` (read before querying metrics)
- Tone of voice: `your tone of voice guide` (read before writing the Slack message)
- Previous prep messages: `your output folder/message-history.md` (for continuity)
- Product strategy context: `your product strategy documentation` (for interpreting metrics)

## Google Drive Folder

**Folder URL:** YOUR_GOOGLE_DRIVE_FOLDER_URL
**Folder ID:** `YOUR_FOLDER_ID`

This folder contains one Google Slides deck per bi-weekly session as a historical archive.

## Slack Channel

**Target channel:** `#monday-campaigns` (ID: `YOUR_TEAM_CHANNEL_ID`)

## Tools

### Google Workspace MCP

| Tool | Purpose |
|------|---------|
| Drive search tool | Find the latest presentation in the bi-weekly folder |
| Slides text extraction tool | Extract text content from the most recent deck |
| Slides metadata tool | Get metadata (title, slide count, last modified) |

### Data Warehouse MCP

| Tool | Purpose |
|------|---------|
| SQL query tool | Query current metrics from your data warehouse |
| Data exploration tool | Explore metrics when the data guide doesn't cover a specific query |

### Slack MCP

| Tool | Purpose |
|------|---------|
| `slack_search_public` | Search for feature, PR, and research discussions |
| Slack message tool | Draft the prep message to #monday-campaigns |

## Workflow

### Step 1 -- Find the latest deck

Search the Google Drive folder for the most recent presentation:

```
Tool: drive_search
Arguments:
  query: "YOUR_GOOGLE_DRIVE_FOLDER_URL"
```

From the results, identify the most recently modified Google Slides file. Note its `presentationId` (the file ID) and title.

If no files are found, try searching by folder name or ID directly. If still nothing, flag the issue and proceed with steps 3-5 using available data.

### Step 2 -- Read the last deck

Extract the content of the most recent presentation:

```
Tool: slides_getMetadata
Arguments:
  presentationId: [file ID from step 1]
```

```
Tool: slides_getText
Arguments:
  presentationId: [file ID from step 1]
```

Parse the extracted text to identify:

- **Section structure:** Which team member owns which section (engineering, design, GTM, product, analyst)
- **Metrics presented:** Any numbers, KPIs, or data points shown (ARR, DAU, campaigns sent, conversion rates, etc.)
- **Topics covered:** Features, releases, research, and initiatives discussed
- **Action items or open threads:** Anything flagged as "next steps" or "to follow up"

Record the metrics values as the "last meeting baseline" for comparison in step 3.

### Step 3 -- Pull current metrics

Read `your data guide` for table schemas and metric definitions.

Query the same metrics identified in the deck (step 2). At minimum, always include:

| Metric | Source |
|--------|--------|
| ARR | data warehouse (your SQL query tool) |
| Paying accounts | data warehouse |
| DAU | data warehouse |
| Product sent (last 14 days) | data warehouse |
| New signups (last 14 days) | data warehouse |

For each metric, calculate:
- **Current value**
- **Value at last meeting** (from step 2, if available)
- **Delta** (absolute and percentage change)

If the last deck's numbers can't be parsed, query the metric for both the current date and 14 days ago to show the 2-week trend.

### Step 4 -- Scan Slack for key discussions

Search Slack for the last 14 days of activity relevant to the bi-weekly update.

**4a. Feature and release discussions:**

```
Tool: slack_search_public
Arguments:
  query: "feature OR release OR shipped OR launched in:#monday-campaigns after:[14 days ago YYYY-MM-DD]"
  sort: "timestamp"
  sort_dir: "desc"
  limit: 20
```

**4b. PR discussions:**

```
Tool: slack_search_public
Arguments:
  query: "PR OR pull request OR merged in:#monday-campaigns after:[14 days ago YYYY-MM-DD]"
  sort: "timestamp"
  sort_dir: "desc"
  limit: 20
```

Also search in other product channels:

```
Tool: slack_search_public
Arguments:
  query: "PR OR pull request OR merged has:link after:[14 days ago YYYY-MM-DD]"
  sort: "timestamp"
  sort_dir: "desc"
  limit: 20
  context_channel_id: "YOUR_TEAM_CHANNEL_ID"
```

**4c. Research and findings:**

```
Tool: slack_search_public
Arguments:
  query: "research OR findings OR analysis OR insight OR experiment in:#monday-campaigns after:[14 days ago YYYY-MM-DD]"
  sort: "timestamp"
  sort_dir: "desc"
  limit: 20
```

From all results, compile:
- **Features shipped or in progress:** What was released, what's close to release
- **PRs of note:** Significant code changes, refactors, or new capabilities
- **Research and insights:** User research findings, data analysis results, experiment outcomes
- **Decisions made:** Important decisions documented in Slack
- **Open questions:** Threads that haven't been resolved

### Step 5 -- Compose and send prep message

Read `your tone of voice guide` before writing.

If `your output folder/message-history.md` exists, read it to maintain continuity and avoid repeating phrasing.

Draft a Slack message for `#monday-campaigns` with this structure:

```
*Bi-weekly update prep* · [meeting date, e.g. Thursday 27 Mar]

Hey team, time to prep for the bi-weekly. Here's where we stand and what to cover.

*Numbers snapshot (vs last meeting):*
• ARR: [current] ([+/- delta])
• Paying accounts: [current] ([+/- delta])
• DAU: [current] ([+/- delta])
• Product sent (14d): [current] ([+/- delta])
• [Any other metrics from the deck]

*Key topics from the last 2 weeks:*
• [Feature/release 1] — [1-line summary]
• [Feature/release 2] — [1-line summary]
• [Research finding] — [1-line summary]
• [PR or technical highlight] — [1-line summary]

*Slides:* [link to Google Drive folder]
Please update your section before [day]. Let me know if anything's unclear or if you need data for your slides.
```

Apply `make-human-lite` (`~/.cursor/skills/make-human-lite/SKILL.md`) as a final step before sending.

Send via draft:

```
Tool: slack_send_message_draft
Arguments:
  channel_id: "YOUR_TEAM_CHANNEL_ID"
  message: [composed message]
```

### Step 6 -- Archive the message

Create the output directory if it doesn't exist.

Append the sent message to `your output folder/message-history.md` in this format:

```markdown
---

## [Meeting date]

### Prep message (#monday-campaigns)
[Full message text as sent]

### Numbers at time of prep
| Metric | Value | vs Last Meeting |
|--------|-------|-----------------|
| [Metric] | [Value] | [Delta] |
```

This creates a running record of all bi-weekly prep communications and the metrics at each point in time.

## Guidelines

- No signature at the end of the Slack message
- Keep the message scannable: team members should understand what's needed in 30 seconds
- Be specific with numbers: "up 12%" not "increased"
- Link directly to the Google Drive folder so people can jump to their slides
- If a metric can't be queried, note it honestly rather than omitting it
- Read the data guide every time: table schemas may have changed
- The tone should be collaborative and casual: this is a team prep message, not a formal update

## Change Log

| Date | Change |
|------|--------|
| 26 Mar 2026 | Initial version |
