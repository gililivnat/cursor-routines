---
name: voice-of-the-customer
description: Aggregates customer insights from seven sources (product usage data, ask-product Slack, churn signals, Reddit forums, analytics platform behaviour, call intelligence insights, internal team discussions) into a themed VoC report. Use when asked for a voice of the customer report, VoC analysis, customer insights roundup, or customer signal review.
---

# Voice of the Customer

Bi-weekly skill that pulls customer signals from multiple sources, synthesises them into themes, and produces an HTML report + a detailed markdown report.

## When to Use

Run this skill bi-weekly or when asked for a customer insights roundup. Not part of the routines system; triggered manually.

## Inputs

- Previous VoC reports: `Output/voice-of-customer/` (read most recent 2-3 for running context)
- Data guide: `your data guide` (read before querying your data warehouse)
- Product strategy: `your product strategy documentation` (for connecting themes to strategy)
- Tone of voice: `your tone of voice guide` (read before writing)

## Tools

### Data Warehouse MCP

| Tool | Purpose |
|------|---------|
| SQL query tool | Query your data warehouse for usage, adoption, and churn data |
| Data exploration tool | Open-ended data exploration |
| Schema explorer | Explore table schemas |

### Slack MCP

| Tool | Purpose |
|------|---------|
| `slack_search_channels` | Find ask-product and product-churns channels |
| `slack_read_channel` | Read messages from channels |
| `slack_read_thread` | Read thread replies |
| `slack_search_public` | Search for customer-related discussions across channels |

### Analytics MCP

| Tool | Purpose |
|------|---------|
| Analytics query tool | Query feature adoption and usage trends |
| Event discovery tool | List available events for discovery |
| Data quality tool | Surface data quality issues and anomalies |
| Session replay tool | Session replays for qualitative behaviour insight |

### Call Intelligence MCP

| Tool | Purpose |
|------|---------|
| Call listing tool | List recent calls with date filtering |
| Call metadata tool | Get call metadata including the call URL (needed for deep links) |
| Call summary tool | AI-generated summary: key points, topics, action items |
| Call search tool | Search calls by host, date range, or workspace |
| Keyword tracker tool | List keyword trackers (competitors, topics, objections) |
| Call transcript tool | Full speaker-attributed transcript with timing (use for notable quotes) |

### Web Search

| Tool | Purpose |
|------|---------|
| Web search | Search Reddit forums for user pain points, trends, and sentiment |

## Data Sources

### 1. Product Usage Data (data warehouse)

Query your data warehouse for the last 14 days. Read `your data guide` first for correct table names and join keys.

**What to query:**
- Feature adoption rates from `FACT_ACCOUNTS_PRODUCTS_METRICS_DAILY` (DAU, MAPP, WAPP trends)
- Campaign volume and type distribution from `STG_MARKETING_CHANNELS_CAMPAIGNS`
- Activation funnel: install to first campaign created to first campaign sent
- Feature-level engagement where available (campaign types, automation usage, template adoption)

**What to look for:**
- Features with growing or declining adoption
- Activation bottlenecks (high install, low send)
- Segments behaving differently (plan tier, company size, region)

### 2. ask-product Slack Channel

Read the last 14 days of messages from `ask-product`.

**Workflow:**
1. `slack_search_channels` with query "ask-product"
2. `slack_read_channel` with 14-day window
3. `slack_read_thread` for threads with multiple replies

**What to extract:**
- Customer questions (what are they confused about?)
- Complaints (what's frustrating them?)
- Feature requests (what do they wish existed?)
- Workarounds (what creative solutions are they building?)

Tag each observation with: topic, sentiment (positive/negative/neutral), frequency (one-off vs recurring).

### 3. Churn Signals (data warehouse + Slack)

**Quantitative (your data warehouse):**
```
SELECT CANCELLATION_REASON, CANCELLATION_REASON_DESC, COUNT(*) as cnt
FROM L4.DIM_ACCOUNT_SUBSCRIPTION_CONTRACTS
WHERE CANCELLATION_DATE >= DATEADD(day, -14, CURRENT_DATE())
GROUP BY 1, 2
ORDER BY cnt DESC
```

Also query `FACT_ACCOUNTS_PRODUCTS_METRICS_DAILY` for churn volume:
```
SELECT DAY, COUNT(*) as churned_accounts
FROM L4.FACT_ACCOUNTS_PRODUCTS_METRICS_DAILY
WHERE PRODUCT = 'marketing'
  AND PRODUCT_ARR_DAILY_CHANGE_TYPE = 'churn'
  AND DAY >= DATEADD(day, -14, CURRENT_DATE())
GROUP BY DAY
ORDER BY DAY
```

**Qualitative (Slack):**
Read `product-churns` channel for the last 14 days. Extract churn stories, reasons, and patterns.

**What to look for:**
- Top cancellation reasons (ranked by volume)
- Changes in churn reasons vs previous report
- Qualitative context from Slack that explains the numbers

### 4. Reddit Community Voice (Web Search)

Search for posts from the last 14 days across these subreddits:

| Subreddit | Focus |
|-----------|-------|
| r/emailmarketing | Email marketing tools, deliverability, best practices |
| r/marketing | General marketing trends, tool recommendations |
| r/smallbusiness | SMB marketing needs, tool choices, pain points |
| r/digitalmarketing | Digital marketing strategy, automation, tools |
| r/MarketingAutomation | Marketing automation platforms, comparisons |

**Search queries (examples):**
- `site:reddit.com r/emailmarketing "email marketing" frustration OR "wish" OR "need" OR "looking for"`
- `site:reddit.com r/marketing "marketing tool" OR "marketing platform" pain OR problem OR issue`
- `site:reddit.com r/smallbusiness "email marketing" OR "marketing automation" recommendation OR advice`
- `site:reddit.com r/MarketingAutomation trend OR AI OR "new feature"`

**What to extract:**
- Unmet needs and pain points with existing tools
- Emerging trends and shifting expectations (especially around AI)
- What features or capabilities marketers value most
- Common frustrations that your product could address

### 5. Analytics Behavioural Data

Use the analytics MCP to query feature-level engagement. Requires `project_id` (check previous reports or ask the user if unknown).

**What to query:**
- `Get-Events` to discover available campaign-related events
- `Run-Query` with `report_type: "insights"` for feature adoption trends (14-day window, daily granularity)
- `Run-Query` with `report_type: "funnels"` for key conversion funnels (e.g. campaign create to send)
- `Get-Issues` to surface data quality anomalies or event drops

**What to look for:**
- Features with rising or falling engagement
- Funnel drop-off points (where do users abandon?)
- Event volume anomalies (sudden spikes or drops)

**Note:** If the analytics project ID is not yet known, use `Get-Events` to discover it or ask the user. Skip this section gracefully if analytics tool is not configured.

### 6. Customer Call Insights (Call Intelligence)

Pull call summaries from the last 14 days to surface recurring themes, objections, and feature discussions from customer and prospect conversations.

**Workflow:**
1. your call listing tool with `fromDateTime` set to 14 days ago
2. your call summary tool for each call (batch in groups to manage volume; prioritise the most recent 15-20 calls)
3. your keyword tracker tool to pull keyword tracker results (competitor mentions, objection tracking, topic trends)
4. For calls with particularly interesting summaries, use your call metadata tool to get the call URL, then your call transcript tool to find the exact quote and its timestamp
5. Construct a deep link for each notable quote: `{call_url}#t={timestamp_in_seconds}` (the timestamp comes from the transcript's timing data)

**What to extract:**
- Recurring customer pain points raised on calls
- Feature requests or product gaps mentioned by customers or prospects
- Competitor mentions and how your product is compared
- Common objections during sales or CS conversations
- Positive feedback and "aha moments" from call summaries
- Action items from calls that signal product needs
- Notable verbatim quotes with call deep links (aim for 2-4 standout quotes per report)

**What to look for:**
- Themes appearing across multiple calls (strong signal)
- Tracker keywords trending up or down vs previous cycles
- Discrepancies between what customers say on calls vs what they write in Slack or support channels

**Note:** If the call intelligence MCP is not configured or returns no calls, skip gracefully and note it in the output.

### 7. Internal Team Voice (Slack)

Search for customer-related discussions across monday Slack channels from the last 14 days.

**Search terms:**
- "customer said", "user feedback", "feature request", "pain point"
- "complained", "asked for", "struggling with", "confused about"
- "churned because", "cancelled because"

```
Tool: slack_search_public
Arguments:
  query: "customer said" OR "user feedback" OR "feature request"
  sort: "timestamp"
  sort_dir: "desc"
```

Focus on monday channels, not general Slack noise. Extract what the team is hearing informally.

## Workflow

### Step 1 -- Read previous reports

Read the most recent 2-3 reports from `Output/voice-of-customer/`. Note:
- Themes flagged previously (are they growing or fading?)
- "Worth watching" items that may have developed
- Open questions from the last cycle

If no previous reports exist, this is the first run. Skip this step.

### Step 2 -- Gather product usage data

Read `your data guide`, then query the data warehouse for the metrics listed in Data Source 1. Focus on 14-day trends and feature-level adoption.

### Step 3 -- Analyse ask-product channel

Read 14 days of messages from `ask-product`. For each customer question, complaint, or request:
- Extract the core observation
- Tag with topic and sentiment
- Note frequency (is this the first time, or a recurring theme?)

### Step 4 -- Pull churn signals

Run the churn queries from Data Source 3. Read `product-churns` Slack channel. Connect quantitative churn data with qualitative feedback.

### Step 5 -- Scan Reddit community

Run web searches from Data Source 4. For each relevant post:
- Note the subreddit, post title, and key insight
- Tag the theme (pain point, trend, feature gap, tool comparison)
- Assess relevance to your product (high/medium/low)

### Step 6 -- Check analytics platform

Query analytics platform per Data Source 5. If not configured, skip and note it in the output.

### Step 7 -- Pull call intelligence insights

Pull call summaries and tracker data per Data Source 6. For each call summary:
- Extract recurring pain points, feature requests, and competitor mentions
- Note sentiment and whether the theme is new or recurring
- Cross-reference tracker keywords with themes from other sources

For the most compelling themes, pull the transcript to find the exact quote. Use your call metadata tool to retrieve the call URL, then construct a deep link: `{call_url}#t={timestamp_seconds}`. Aim for 2-4 linked quotes that best illustrate the key themes.

If the call intelligence MCP is not configured, skip and note it in the output.

### Step 8 -- Search internal team voice

Search Slack per Data Source 7. Extract what the team is hearing from customers informally.

### Step 9 -- Synthesise into themes

Cross-reference all sources. Group observations into themes using affinity mapping.

For each theme, assess:

| Dimension | Scale |
|-----------|-------|
| **Signal strength** | How many sources confirm it? (1 source = weak, 3+ = strong) |
| **Sentiment** | Positive / Negative / Neutral |
| **Trend direction** | Growing / Stable / Declining (vs previous report) |
| **Actionability** | Can we act on this now? What would we do? |

Connect themes to product strategy from `your product strategy documentation`.

Identify:
- **Converging signals** -- the same theme appearing across multiple sources
- **Emerging needs** -- new pain points or expectations not previously seen
- **Positive signals** -- features or capabilities customers love
- **Strategic gaps** -- areas where our product falls short of market expectations

## Output

### HTML Report

Save as a standalone HTML file to `Output/voice-of-customer/voc-report-YYYY-MM-DD.html`. Use the routine HTML template from `skills/system/routine-html-template.md`.

Open the file with `open` (macOS) so it launches in the browser.

#### HTML structure

```html
<div class="header">
  <h1>🎙️ Voice of the Customer</h1>
  <div class="subtitle">[date range] · [N] sources analysed</div>
</div>

<!-- Key Themes -->
<div class="section">
  <div class="section-title">Key Themes</div>

  <!-- Strong signal (3+ sources) -->
  <div class="card urgent">
    <div class="card-label urgent">[Theme name] · Strong signal</div>
    <div class="card-title">[One-line summary]</div>
    <div class="card-body">[Evidence from multiple sources. Be specific.]</div>
    <div class="card-meta">Sources: [list sources] · Trend: [growing/stable/declining]</div>
  </div>

  <!-- Medium signal (2 sources) -->
  <div class="card warning">
    <div class="card-label warning">[Theme name] · Medium signal</div>
    <div class="card-title">[Summary]</div>
    <div class="card-body">[Evidence]</div>
    <div class="card-meta">Sources: [list] · Trend: [direction]</div>
  </div>

  <!-- Weak/emerging signal (1 source) -->
  <div class="card highlight">
    <div class="card-label highlight">[Theme name] · Emerging</div>
    <div class="card-title">[Summary]</div>
    <div class="card-body">[Evidence]</div>
    <div class="card-meta">Source: [source] · First flagged: [this report / previous date]</div>
  </div>

  <!-- Positive signal -->
  <div class="card success">
    <div class="card-label success">[Theme name] · Positive</div>
    <div class="card-title">[What customers love]</div>
    <div class="card-body">[Evidence]</div>
  </div>
</div>

<hr class="divider">

<!-- Usage Snapshot -->
<div class="section">
  <div class="section-title">Usage Snapshot · Last 14 days</div>

  <div class="metric-grid">
    <div class="metric-card">
      <div class="metric-value">[N]</div>
      <div class="metric-label">DAU (avg)</div>
      <div class="metric-change [up/down/flat]">[+/- %] vs prior 14d</div>
    </div>
    <div class="metric-card">
      <div class="metric-value">[N]</div>
      <div class="metric-label">Product sent/day</div>
      <div class="metric-change [up/down/flat]">[change]</div>
    </div>
    <div class="metric-card">
      <div class="metric-value">[N]</div>
      <div class="metric-label">MAPP</div>
      <div class="metric-change [up/down/flat]">[change]</div>
    </div>
    <div class="metric-card">
      <div class="metric-value">[%]</div>
      <div class="metric-label">Activation rate</div>
      <div class="metric-change [up/down/flat]">[change]</div>
    </div>
  </div>

  <!-- Feature adoption highlights -->
  <div class="card highlight">
    <div class="card-label highlight">Feature adoption</div>
    <div class="card-title">[Key finding about feature usage]</div>
    <div class="card-body">[Numbers and context]</div>
  </div>
</div>

<hr class="divider">

<!-- Customer Voice (ask-product) -->
<div class="section">
  <div class="section-title">Customer Voice · #ask-product</div>

  <div class="card highlight">
    <div class="card-label highlight">[Category: e.g. Feature requests]</div>
    <div class="card-title">[Top theme from the channel]</div>
    <div class="card-body">[Summary with examples. N messages about this topic.]</div>
  </div>

  <div class="card warning">
    <div class="card-label warning">[Category: e.g. Complaints]</div>
    <div class="card-title">[Recurring frustration]</div>
    <div class="card-body">[Detail]</div>
  </div>

  <p class="quiet">[N] messages reviewed · [date range]</p>
</div>

<hr class="divider">

<!-- Churn Signals -->
<div class="section">
  <div class="section-title">Churn Signals</div>

  <div class="metric-grid">
    <div class="metric-card">
      <div class="metric-value">[N]</div>
      <div class="metric-label">Accounts churned</div>
      <div class="metric-change [up/down/flat]">[vs prior 14d]</div>
    </div>
    <div class="metric-card">
      <div class="metric-value">[reason]</div>
      <div class="metric-label">Top reason</div>
      <div class="metric-change flat">[N] accounts</div>
    </div>
  </div>

  <table>
    <thead>
      <tr><th>Reason</th><th>Count</th><th>Trend</th></tr>
    </thead>
    <tbody>
      <tr>
        <td>[Reason]</td>
        <td>[N]</td>
        <td><span class="badge [green/red/amber]">[up/down/stable]</span></td>
      </tr>
    </tbody>
  </table>

  <!-- Qualitative churn context from Slack -->
  <div class="card warning">
    <div class="card-label warning">From #product-churns</div>
    <div class="card-title">[Pattern or notable churn story]</div>
    <div class="card-body">[Detail]</div>
  </div>
</div>

<hr class="divider">

<!-- Community Pulse (Reddit) -->
<div class="section">
  <div class="section-title">Community Pulse · Reddit</div>

  <div class="card highlight">
    <div class="card-label highlight">[Theme]</div>
    <div class="card-title">[Key finding]</div>
    <div class="card-body">[Summary of posts and sentiment. Mention subreddits.]</div>
  </div>

  <p class="quiet">[N] relevant posts found · [subreddits searched]</p>
</div>

<hr class="divider">

<!-- Behavioural Insights (analytics platform) -->
<div class="section">
  <div class="section-title">Behavioural Insights · Analytics</div>

  <div class="card highlight">
    <div class="card-label highlight">[Finding type: e.g. Funnel drop-off]</div>
    <div class="card-title">[Key behavioural insight]</div>
    <div class="card-body">[Numbers and context]</div>
  </div>

  <p class="quiet">If analytics tool not configured: "analytics data not available for this cycle."</p>
</div>

<hr class="divider">

<!-- Customer Call Insights (Call Intelligence) -->
<div class="section">
  <div class="section-title">Customer Call Insights · Call Intelligence</div>

  <div class="metric-grid">
    <div class="metric-card">
      <div class="metric-value">[N]</div>
      <div class="metric-label">Calls reviewed</div>
    </div>
    <div class="metric-card">
      <div class="metric-value">[N]</div>
      <div class="metric-label">Unique themes</div>
    </div>
  </div>

  <div class="card highlight">
    <div class="card-label highlight">[Theme from calls]</div>
    <div class="card-title">[Key finding]</div>
    <div class="card-body">[Summary with evidence from call summaries. Mention frequency across calls.]</div>
  </div>

  <div class="card warning">
    <div class="card-label warning">Competitor mentions</div>
    <div class="card-title">[Most-mentioned competitor or comparison theme]</div>
    <div class="card-body">[What's being said, tracker data if available]</div>
  </div>

  <!-- Notable quotes with deep links -->
  <div class="card">
    <div class="card-label">Notable quotes</div>
    <div class="card-body">
      <blockquote>"[Verbatim quote from customer]"<br>
        <span class="quiet">— [Speaker], <a href="[call_url]#t=[timestamp_seconds]">[Call title, date]</a></span>
      </blockquote>
      <blockquote>"[Another quote]"<br>
        <span class="quiet">— [Speaker], <a href="[call_url]#t=[timestamp_seconds]">[Call title, date]</a></span>
      </blockquote>
    </div>
  </div>

  <p class="quiet">If call intelligence not configured: "call intelligence data not available for this cycle."</p>
</div>

<hr class="divider">

<!-- Strategic Implications -->
<div class="section">
  <div class="section-title">Strategic Implications</div>

  <div class="card urgent">
    <div class="card-label urgent">Act now</div>
    <div class="card-title">[What needs attention]</div>
    <div class="card-body">[Detail and recommended action]</div>
  </div>

  <div class="card success">
    <div class="card-label success">Opportunity</div>
    <div class="card-title">[What we could capitalise on]</div>
    <div class="card-body">[Detail]</div>
  </div>

  <div class="card warning">
    <div class="card-label warning">Worth watching</div>
    <div class="card-title">[Signal to monitor next cycle]</div>
    <div class="card-body">[Why and what to look for]</div>
  </div>
</div>
```

Use card types based on signal strength:
- `card urgent` -- strong signals (3+ sources) or issues needing immediate attention
- `card warning` -- medium signals (2 sources) or emerging concerns
- `card highlight` -- single-source observations or informational findings
- `card success` -- positive signals, things working well

If a section has no findings, use: `<p class="quiet">No notable findings this cycle.</p>`

### Markdown Report

Save to `Output/voice-of-customer/voc-report-YYYY-MM-DD.md`:

```markdown
# Voice of the Customer -- [Date Range]

## Executive Summary

[3-5 sentences. What are customers telling us right now? Lead with the strongest
theme. Connect to strategy and previous reports.]

## Key Themes

### [Theme 1: Strongest signal]
- **Signal strength:** [Strong/Medium/Weak] ([N] sources)
- **Sentiment:** [Positive/Negative/Neutral]
- **Trend:** [Growing/Stable/Declining]
- **Evidence:**
  - Usage data: [finding]
  - ask-product: [finding]
  - Reddit: [finding]
  - Churn: [finding]
  - analytics platform: [finding]
  - Internal: [finding]
- **Strategic connection:** [How this relates to product strategy]
- **Recommended action:** [What we should do]

### [Theme 2]
...

## Usage Data

### Headline metrics (14-day average)
| Metric | Value | vs prior 14d | Trend |
|--------|-------|--------------|-------|
| DAU | [N] | [change] | [direction] |
| MAPP | [N] | [change] | [direction] |
| Product sent/day | [N] | [change] | [direction] |
| Activation rate | [%] | [change] | [direction] |

### Feature adoption highlights
- [Finding with numbers]

## Customer Feedback (ask-product)

### [Category]
- [Observation] -- [frequency: one-off/recurring] -- [sentiment]

## Churn Analysis

### Top cancellation reasons (14 days)
| Reason | Count | vs prior period |
|--------|-------|-----------------|
| [reason] | [N] | [change] |

### Qualitative context
- [Insight from product-churns Slack]

## Community Voice (Reddit)

### [Theme]
- [Finding] -- [Subreddit] -- [Relevance to your product]

## Behavioural Insights (analytics platform)

- [Finding with numbers]

## Customer Call Insights (Call Intelligence)

### Call themes (14 days, [N] calls reviewed)
- [Recurring theme] -- [N] calls -- [sentiment]

### Competitor mentions
- [Competitor] -- [context of how they're compared] -- [N] mentions

### Notable quotes
> "[Verbatim quote]"
> — [Speaker], [Call title, date] ([listen in call recording]({call_url}#t={timestamp_seconds}))

### Notable signals
- [Insight from call summaries or tracker data]

## Internal Team Voice

- [What the team is hearing] -- [Channel] -- [Date]

## Patterns & Trends

- [Pattern] -- [Connection to previous reports]
- [Emerging signal] -- [First flagged: date, or new this cycle]

## Strategic Implications

- **Act now:** [Issues needing immediate attention]
- **Opportunity:** [Gaps we could fill or strengths to amplify]
- **Worth watching:** [Signals to monitor in the next cycle]

## Open Questions

- [Questions to investigate next time]

---

*Previous report:* [link to previous report file, or "First report"]
*Sources analysed:* [date range] · your data warehouse, ask-product, product-churns, Reddit, analytics platform, call intelligence, Internal Slack
*Freshness:* Research conducted [date]
```

## Guidelines

- British English throughout
- No em dashes (use commas, colons, or "–")
- Theme-first structure: organise by customer theme, not by data source
- Be specific with numbers: "12 messages about template limitations" not "many users mentioned templates"
- Cross-reference sources: a theme backed by 3 sources is stronger than one backed by 1
- Flag when a theme from a previous report has grown, faded, or been resolved
- The HTML report should be scannable in ~90 seconds. The markdown report has the depth
- Read `your tone of voice guide` before writing
- If a data source is unavailable (e.g. analytics tool not configured), skip gracefully and note it

## Change Log

| Date | Change |
|------|--------|
| 18 Mar 2026 | Initial version. Six data sources: product usage (data warehouse), ask-product (Slack), churn signals (data warehouse + Slack), Reddit (web search), analytics platform (behavioural), internal team voice (Slack). Bi-weekly cadence, standalone skill. |
| 31 Mar 2026 | Added call intelligence platform as 7th data source. Pulls call summaries and keyword tracker data to surface recurring themes, competitor mentions, and objections from customer conversations. |
