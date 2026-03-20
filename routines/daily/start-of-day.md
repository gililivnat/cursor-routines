# Start of Day

Run these skills in order at the start of each working day. Each step reads its SKILL.md, executes, and updates `state.md` when done.

---

## Steps

### 1. Review Slack DMs

**Skill:** `skills/review-slack-dms/SKILL.md`
**State field:** `daily_start.review_slack_dms`

Read personal Slack direct messages from the last 24 hours. Flag anything important, summarise key conversations.

---

### 2. Review Product Channels

**Skill:** `skills/review-campaigns-channels/SKILL.md`
**State field:** `daily_start.review_campaigns_channels`

Find all Slack channels with "Product" in the name. Read recent messages, summarise activity and flag important threads.

---

### 3. Summarise Slack + Suggest Tasks

**Skill:** `skills/summarise-slack-and-suggest-tasks/SKILL.md`
**State field:** `daily_start.summarise_slack_suggest_tasks`

Consolidate findings from steps 1 and 2. Produce a unified summary and suggest actionable tasks.

**Depends on:** Steps 1 and 2 must be complete first.

---

### 4. Review Daily Metrics

**Skill:** `skills/review-daily-metrics/SKILL.md`
**State field:** `daily_start.review_daily_metrics`

Query main product metrics and surface highlights, anomalies, and trends.

---

### 5. Weekly Competitor Intel (Monday only)

**Skill:** `skills/weekly-competitor-intel/SKILL.md`
**State field:** `daily_start.weekly_competitor_intel`
**Condition:** Only runs on Mondays. On other days, treat as already complete (skip).

Gathers competitive intelligence from Reddit marketing forums, competitor blogs/changelogs, the `campaign-competitor-updates` Slack channel, and internal Slack discussions. Appends a "Competitor Intel" section to the morning briefing HTML. Saves a full markdown report to `your output folder/`.

**Depends on:** Steps 1-4 must be complete first (the morning briefing HTML must exist before appending).

---

### 6. Review Gmail

**Skill:** `skills/review-gmail/SKILL.md`
**State field:** `daily_start.review_gmail`

Scan Gmail inbox for urgent or important emails from the last 24 hours. Append a Gmail section to the morning briefing HTML with direct links to each email.

---

### 7. Review Upcoming Meetings

**Skill:** `skills/review-upcoming-meetings/SKILL.md`
**State field:** `daily_start.review_upcoming_meetings`

Review today's calendar using Google Workspace MCP. Highlight important meetings, prepare 3-4 lines of context or talking points for each. Detect customer meetings (external attendees) and offer to run the `prepare-customer-interview` skill for those accounts. Appends a Calendar section to the morning briefing HTML.

---

## Completion

When all active steps are done, the orchestrator sets `daily_start.flag` to `1` in `state.md`.
