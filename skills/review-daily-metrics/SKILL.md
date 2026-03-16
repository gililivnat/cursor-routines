---
name: review-daily-metrics
description: Queries main product metrics and surfaces highlights, anomalies, and trends. Use as part of the daily start-of-day routine.
---

# Review Daily Metrics

## When to Use

This skill is called by the start-of-day routine (step 4). It can also be run independently when asked for a quick metrics check.

## Inputs

- Data guide: `your data guide` (read before querying)
- Product strategy context: `your product strategy documentation` (for interpreting metrics)

## Tools

### Data Warehouse MCP

| Tool | Purpose |
|------|---------|
| SQL query tool | Run SQL against Snowflake for specific metrics |
| Data exploration tool | AI agent for open-ended data exploration |
| Schema explorer | Explore table schemas and column descriptions |

## Metrics to Track

These are the default daily metrics. This can be refined this list over time.

| Metric | What it measures | Why it matters |
|--------|-----------------|----------------|
| DAU | Daily active users | Product health pulse |
| Core actions (last 24h) | Volume of primary product actions | Core product usage |
| Creation rate | New items/objects created | Engagement signal |
| New signups (last 24h) | New accounts entering the product | Growth indicator |
| Activation rate | Signup to first meaningful action | Onboarding health |
| Conversion to paid | Trial to paid ratio | Revenue pipeline |

**Note:** The specific SQL queries depend on table schemas documented in `your data guide`. Always read the data guide before writing queries to use correct table names, join keys, and known gotchas.

## Workflow

### Step 1 -- Read the data guide

Read your data guide/schema documentation for current table schemas, metric definitions, and known data issues.

### Step 2 -- Query metrics

For each metric in the table above, write and execute a SQL query. Compare today's value against:
- Yesterday (daily change)
- Same day last week (weekly trend)
- 7-day rolling average (smoothed trend)

Use your SQL query tool for well-defined queries. Use your data exploration tool if you need to explore or the data guide doesn't cover a specific metric.

### Step 3 -- Identify highlights

Flag anything notable:
- **Spikes or drops** > 15% compared to the 7-day average
- **Trend changes** (e.g. 3+ consecutive days of decline)
- **Records** (highest/lowest in the past 30 days)
- **Anomalies** (unexpected patterns, data gaps)

### Step 4 -- Produce summary

Write a concise metrics summary with highlights.

## Output

Generate HTML content using the routine template at `skills/system/routine-html-template.md`. Read that file for the full CSS and component patterns.

**If running as part of the start-of-day routine:** append this section to the morning briefing HTML (after the Slack sections but before the footer). The `summarise-slack-and-suggest-tasks` skill assembles the combined file.

**If running standalone:** save as a standalone HTML file to `your output folder/daily-metrics-YYYY-MM-DD.html` and open it.

### HTML structure for this skill

```html
<hr class="divider">

<div class="section">
  <div class="section-title">Daily Metrics · [date]</div>

  <div class="metric-grid">
    <div class="metric-card">
      <div class="metric-value">[N]</div>
      <div class="metric-label">DAU</div>
      <div class="metric-change [up/down/flat]">[+/- %] vs 7d avg</div>
    </div>
    <div class="metric-card">
      <div class="metric-value">[N]</div>
      <div class="metric-label">Product sent</div>
      <div class="metric-change [up/down/flat]">[+/- %] vs yesterday</div>
    </div>
    <div class="metric-card">
      <div class="metric-value">[N]</div>
      <div class="metric-label">New signups</div>
      <div class="metric-change [up/down/flat]">[+/- %] vs 7d avg</div>
    </div>
    <div class="metric-card">
      <div class="metric-value">[%]</div>
      <div class="metric-label">Activation rate</div>
      <div class="metric-change [up/down/flat]">[+/- pp] vs 7d avg</div>
    </div>
    <div class="metric-card">
      <div class="metric-value">[%]</div>
      <div class="metric-label">Conversion to paid</div>
      <div class="metric-change [up/down/flat]">[+/- pp] vs 7d avg</div>
    </div>
  </div>
</div>

<div class="section">
  <div class="section-title">Highlights</div>
  <div class="card success">
    <div class="card-title">[Notable positive finding]</div>
    <div class="card-body">[Detail with numbers]</div>
  </div>
  <div class="card warning">
    <div class="card-label warning">Worth watching</div>
    <div class="card-title">[Emerging trend or anomaly]</div>
    <div class="card-body">[Detail]</div>
  </div>
</div>
```

If any metric can't be queried (missing table, data gap), note it with: `<p class="quiet">Could not query [metric]: [reason]</p>`

## Guidelines

- Lead with the most important highlight, not the full table
- Be specific with numbers: "up 12%" not "increased"
- Flag data quality issues honestly (small samples, missing days, known pipeline delays)
- Read the data guide every time. Table schemas may have changed since the skill was written
- Keep the summary scannable. the user should understand the picture in 15 seconds

## Change Log

| Date | Change |
|------|--------|
| 13 Mar 2026 | Initial version. Customise the Metrics to Track table for your product. |
