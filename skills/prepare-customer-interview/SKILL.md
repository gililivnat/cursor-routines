---
name: prepare-customer-interview
description: Researches a customer before an interview by pulling usage data, financial info, feedback, and company context from multiple sources. Outputs a short summary and an HTML dossier. Use when asked to prepare for a customer interview, research a customer, or build a customer profile.
---

# Prepare Customer Interview

## When to Use

Run this skill when asked to:
- Prepare for a customer interview
- Research a customer account
- Build a customer dossier or profile
- Get background on a specific account before a call

## Inputs

| Input | Required | Notes |
|-------|----------|-------|
| Customer identifier | Yes | Can be an account ID (`PULSE_ACCOUNT_ID`), company name, or email/domain |
| Website URL | Optional | If not provided, attempt to infer from the domain or ask the user |

If the identifier is ambiguous or returns multiple matches, present the options and ask the user to confirm.

## Tools

### Data Warehouse MCP

| Tool | Purpose |
|------|---------|
| SQL query tool | Run SQL against your data warehouse for account data, campaigns, financials |
| Data exploration tool | Explore data when schemas are unclear or for open-ended questions |

### Slack MCP

| Tool | Purpose |
|------|---------|
| `slack_search_public` | Search channels for customer mentions, feedback, support threads |
| `slack_search_channels` | Find relevant channels by name |
| `slack_read_channel` | Read channel messages for context |
| `slack_read_thread` | Read thread replies for full conversations |

### Analytics MCP

| Tool | Purpose |
|------|---------|
| Analytics query tool | Query user-level behavioural events (best effort) |

### Web

| Tool | Purpose |
|------|---------|
| `WebFetch` | Pull the customer's website to understand their business |

### Monday API MCP (your meeting notetaker MCP)

| Tool | Purpose |
|------|---------|
| `search` | Search for support boards or tickets mentioning the customer (best effort) |

## Workflow

### Step 1 -- Read the data guide

Read `your data guide` for current table schemas, join keys, and known gotchas.

### Step 2 -- Resolve the account

Based on the identifier type:

**Account ID:** Use directly as `PULSE_ACCOUNT_ID`.

**Company name:** Query your data warehouse to find matching accounts:

```sql
SELECT PULSE_ACCOUNT_ID, PRODUCT, FIRST_PRODUCT_INSTALL_DATE,
       IS_PAYING_PRODUCT, ACCOUNT_INFRA_REGION
FROM BIGBRAIN.L4.DIM_ACCOUNTS_PRODUCTS_METRICS
WHERE PRODUCT = 'marketing'
  AND LOWER(PULSE_ACCOUNT_ID::VARCHAR) LIKE '%{search_term}%'
```

If no results, use your data exploration tool to search across account tables for the company name.

**Email/domain:** Extract the domain and search. Also use the domain to construct the website URL (`https://{domain}`).

If multiple accounts match, present them to the user and ask which one to research.

### Step 3 -- Pull product usage data

Query product activity for the resolved account:

```sql
SELECT c.ID, c.NAME, c.TYPE, c.STATUS, c.SENTON, c.CREATEDAT
FROM BIGBRAIN.STG.STG_MARKETING_CHANNELS_CAMPAIGNS c
WHERE c.ACCOUNTID = {account_id}
  AND c.SOURCE != 'IL'
ORDER BY c.CREATEDAT DESC
```

Then pull campaign performance metrics:

```sql
SELECT s.CAMPAIGNID, s.TOTALSENT, s.TOTALDELIVERED,
       s.TOTALOPENS, s.UNIQUEOPENS, s.TOTALCLICKS, s.UNIQUECLICKS
FROM BIGBRAIN.L1.MARKETING_CHANNELS_EMAILCAMPAIGNANALYTICSSUMMARY_US s
WHERE s.ACCOUNTID = {account_id}
```

Repeat for EU and AU regions if the account's region warrants it.

Summarise:
- Total campaigns created vs sent
- Campaign types used
- Send volumes and frequency (last 30d, last 90d, all time)
- Performance: open rate, click rate, bounce rate
- Most recent campaign date

### Step 4 -- Pull financial data

```sql
SELECT DAY, PLAN_TIER, PLAN_PERIOD, SEATS, BILLING_ARR, PRODUCT_ARR,
       PRODUCT_ARR_DAILY_CHANGE, PRODUCT_ARR_DAILY_CHANGE_TYPE,
       DAU, MAPP, IS_PAYING_PRODUCT, IS_IN_TRIAL
FROM BIGBRAIN.L4.FACT_ACCOUNTS_PRODUCTS_METRICS_DAILY
WHERE PULSE_ACCOUNT_ID = {account_id}
  AND PRODUCT = 'marketing'
ORDER BY DAY DESC
LIMIT 90
```

Also pull subscription lifecycle:

```sql
SELECT EVENT_TYPE, EVENT_TIMESTAMP, CONTRACT_STATUS,
       CANCELLATION_REASON, CANCELLATION_REASON_DESC,
       ARR, NEXT_ARR, PLAN_TIER, PERIOD,
       RENEWAL_STATUS, CANCEL_ON_RENEWAL_IND
FROM BIGBRAIN.L4.DIM_ACCOUNT_SUBSCRIPTION_CONTRACTS
WHERE PULSE_ACCOUNT_ID = {account_id}
ORDER BY EVENT_TIMESTAMP DESC
```

Summarise:
- Current plan tier, period, seats, ARR
- Billing history (upgrades, downgrades, churn events)
- Trial status and dates
- Cancellation signals (cancel-on-renewal, churn reason if any)
- MAPP and DAU trend (last 30 days)

### Step 5 -- Pull other monday products

```sql
SELECT PRODUCT, FIRST_PRODUCT_INSTALL_DATE, IS_PAYING_PRODUCT,
       FIRST_BILLING_ARR_DATE, FIRST_WEEK_DAU_7,
       IS_SUCCESSFUL_SETUP, CHANNEL, SIGNUP_FLOW
FROM BIGBRAIN.L4.DIM_ACCOUNTS_PRODUCTS_METRICS
WHERE PULSE_ACCOUNT_ID = {account_id}
ORDER BY IS_PAYING_PRODUCT DESC, FIRST_PRODUCT_INSTALL_DATE
```

Summarise:
- Which monday products are installed and active
- Which are paid vs free
- Tenure per product
- Engagement level (first week DAU as proxy)

### Step 6 -- Search for customer feedback (Slack)

Search `campaigns-stats` and other relevant channels for mentions of the customer:

1. Use `slack_search_public` with the company name as the query
2. Also search with the account ID if available
3. Check `campaigns-stats`, `ask-product`, and `product-churns` channels
4. For relevant results, read the full thread with `slack_read_thread`

Summarise:
- Feedback themes (positive, negative, feature requests)
- Support issues raised
- Direct quotes worth noting
- Links to relevant Slack threads

### Step 7 -- Search for support tickets (best effort)

Use your meeting notetaker MCP `search` tool to look for the company name across monday boards. If a support or tickets board exists, pull relevant items.

If nothing is found, note: "No dedicated support ticket system accessible. Slack search was used as the primary source for support-related signals."

### Step 8 -- Pull company website

Use `WebFetch` to retrieve the customer's website. Extract:
- What the company does (industry, product/service)
- Company size indicators (team page, about page)
- Target market
- Any marketing-related context (do they have a blog, newsletter, social presence?)

If the website URL is unknown, ask the user to provide it.

### Step 9 -- Compile and generate output

#### Short summary (in conversation)

Present 4-6 bullet points covering:
- Who they are (company, industry, size)
- How they use campaigns (volume, types, performance)
- Financial status (plan, ARR, tenure, risk signals)
- Other monday products in use
- Notable feedback or support signals
- One or two suggested areas to probe in the interview

#### HTML dossier

Generate an HTML file using the routine template at `skills/system/routine-html-template.md`. Read that file for the full CSS and component patterns.

**Save to:** `your output folder/customer-interview-prep-{company-slug}-YYYY-MM-DD.html`
**Open with:** `open` command

### HTML Structure

```html
<div class="header">
  <h1>Customer Interview Prep</h1>
  <div class="subtitle">{Company Name} · {date}</div>
</div>

<!-- Company Overview -->
<div class="section">
  <div class="section-title">Company Overview</div>
  <div class="card">
    <div class="card-body">[What the company does, industry, size, target market]</div>
  </div>
</div>

<!-- Account Snapshot -->
<div class="section">
  <div class="section-title">Account Snapshot</div>
  <div class="metric-grid">
    <div class="metric-card">
      <div class="metric-value">[tier]</div>
      <div class="metric-label">Plan</div>
    </div>
    <div class="metric-card">
      <div class="metric-value">$[N]</div>
      <div class="metric-label">ARR</div>
    </div>
    <div class="metric-card">
      <div class="metric-value">[N]</div>
      <div class="metric-label">Seats</div>
    </div>
    <div class="metric-card">
      <div class="metric-value">[N] months</div>
      <div class="metric-label">Tenure</div>
    </div>
    <div class="metric-card">
      <div class="metric-value">[region]</div>
      <div class="metric-label">Region</div>
    </div>
  </div>
</div>

<!-- Product Usage -->
<div class="section">
  <div class="section-title">Product Usage</div>
  <div class="metric-grid">
    <div class="metric-card">
      <div class="metric-value">[N]</div>
      <div class="metric-label">Product created</div>
    </div>
    <div class="metric-card">
      <div class="metric-value">[N]</div>
      <div class="metric-label">Product sent</div>
    </div>
    <div class="metric-card">
      <div class="metric-value">[N]</div>
      <div class="metric-label">Total emails sent</div>
    </div>
    <div class="metric-card">
      <div class="metric-value">[%]</div>
      <div class="metric-label">Avg open rate</div>
    </div>
    <div class="metric-card">
      <div class="metric-value">[%]</div>
      <div class="metric-label">Avg click rate</div>
    </div>
  </div>
  <table>
    <thead><tr><th>Period</th><th>Product sent</th><th>Emails sent</th><th>Open rate</th><th>Click rate</th></tr></thead>
    <tbody>
      <tr><td>Last 30 days</td><td>[N]</td><td>[N]</td><td>[%]</td><td>[%]</td></tr>
      <tr><td>Last 90 days</td><td>[N]</td><td>[N]</td><td>[%]</td><td>[%]</td></tr>
      <tr><td>All time</td><td>[N]</td><td>[N]</td><td>[%]</td><td>[%]</td></tr>
    </tbody>
  </table>
  <div class="card">
    <div class="card-title">Recent campaigns</div>
    <div class="card-body">[List of last 5 campaigns with name, date, type, status]</div>
  </div>
</div>

<!-- Other Monday Products -->
<div class="section">
  <div class="section-title">Other Monday Products</div>
  <table>
    <thead><tr><th>Product</th><th>Installed</th><th>Paying</th><th>First week DAU</th></tr></thead>
    <tbody>
      <tr><td>[product]</td><td>[date]</td><td>[yes/no]</td><td>[N]</td></tr>
    </tbody>
  </table>
</div>

<!-- Financial History -->
<div class="section">
  <div class="section-title">Financial History</div>
  <div class="card [success/warning/urgent]">
    <div class="card-title">[Current status summary]</div>
    <div class="card-body">[Billing events timeline: upgrades, downgrades, churn signals]</div>
  </div>
</div>

<!-- Customer Voice -->
<div class="section">
  <div class="section-title">Customer Voice</div>
  <!-- One card per feedback source or theme -->
  <div class="card highlight">
    <div class="card-label highlight">campaigns-stats</div>
    <div class="card-body">[Feedback summary with quotes]</div>
    <div class="card-meta">[Link to Slack thread]</div>
  </div>
  <div class="card">
    <div class="card-label">ask-product</div>
    <div class="card-body">[Support questions or issues raised]</div>
    <div class="card-meta">[Link to Slack thread]</div>
  </div>
</div>

<!-- Suggested Interview Questions -->
<div class="section">
  <div class="section-title">Suggested Interview Questions</div>
  <ol class="priority-list">
    <li>
      <div class="card-title">[Question based on data gap or interesting signal]</div>
      <div class="card-body">[Why this question matters, what data prompted it]</div>
    </li>
  </ol>
</div>
```

## Guidelines

- Lead with the most important signals, not exhaustive data dumps
- Be specific with numbers: "sent 47 campaigns in the last 90 days" not "active user"
- If a data source returns nothing, include an empty-state note in the HTML rather than omitting the section
- Flag data quality issues honestly (region gaps, missing tables, small samples)
- Suggested interview questions should be grounded in the data, not generic
- Keep the summary scannable. the user should understand the picture in 30 seconds
- Preserve Slack thread links so the user can jump to the original conversation

## Change Log

| Date | Change |
|------|--------|
| 18 Mar 2026 | Initial version. Sources: your data warehouse (campaigns, financials, product metrics), Slack (campaigns-stats, ask-product, churns), website, analytics platform (best effort), monday boards (best effort). |
