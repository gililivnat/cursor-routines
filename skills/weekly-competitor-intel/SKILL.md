---
name: weekly-competitor-intel
description: Gathers competitive intelligence from Reddit forums, competitor blogs/changelogs, the campaign-competitor-updates Slack channel, and internal discussions. Appends a competitor section to the morning briefing HTML and saves a full markdown report. Use as part of the Monday start-of-day routine (step 5).
---

# Weekly Competitor Intel

## When to Use

This skill is called by the start-of-day routine (step 5) **on Mondays only**, after `review-daily-metrics` has completed. It can also be run independently when asked for a competitive update.

## Inputs

- Previous competitor reports: `your output folder/` (read most recent first for running context)
- Product strategy context: `your product strategy documentation`
- Builders 2.0 methodology: `Tools/External_skills/boards_builders_2_0/skills/product-management/analyzing-competitors/SKILL.md`

## Tools

### Web Search

| Tool | Purpose |
|------|---------|
| Web search | Search Reddit forums, competitor blogs, changelogs, and news for recent developments |

### Slack MCP

| Tool | Purpose |
|------|---------|
| `slack_get_channel_messages` | Read recent messages from `campaign-competitor-updates` |
| `slack_search_public` | Search for competitive intelligence discussions across public channels |
| `slack_list_channels` | Find the `campaign-competitor-updates` channel ID |

## Data Sources

### 1. Reddit Marketing Forums

Search for recent posts (last 7 days) in these subreddits:

| Subreddit | Focus |
|-----------|-------|
| r/emailmarketing | Email marketing tools, deliverability, best practices |
| r/marketing | General marketing trends, tool recommendations |
| r/smallbusiness | SMB marketing needs, tool choices, pain points |
| r/digitalmarketing | Digital marketing strategy, automation, tools |
| r/MarketingAutomation | Marketing automation platforms, comparisons |

**What to search for:**
- Mentions of competitors: Mailchimp, HubSpot, ActiveCampaign, Brevo, Constant Contact
- Tool comparison threads ("best email marketing tool", "Mailchimp alternative", "switching from X to Y")
- Feature requests and pain points with existing tools
- Emerging tools or platforms gaining attention
- Marketing trends relevant to the SMB segment

**Search queries (examples):**
- `site:reddit.com r/emailmarketing "mailchimp" OR "hubspot" OR "activecampaign" OR "brevo" OR "constant contact"`
- `site:reddit.com r/marketing "email marketing tool" OR "marketing automation"`
- `site:reddit.com r/smallbusiness "email marketing" OR "marketing platform"`

### 2. Competitor Blogs and Changelogs

Search for recent posts from each competitor. Focus on product announcements, feature releases, and strategy signals.

| Competitor | What to search |
|------------|---------------|
| **Mailchimp** | `site:mailchimp.com/resources OR site:mailchimp.com/help/whats-new` + general web search for "Mailchimp new feature" or "Mailchimp update" |
| **HubSpot** | `site:blog.hubspot.com` or `site:hubspot.com/changelog` + web search for "HubSpot Marketing Hub update" |
| **ActiveCampaign** | `site:activecampaign.com/blog` + web search for "ActiveCampaign new feature" |
| **Brevo** | `site:brevo.com/blog` + web search for "Brevo update" or "Brevo new feature" |
| **Constant Contact** | `site:constantcontact.com/blog` + web search for "Constant Contact update" |

**What to look for:**
- New feature launches or beta announcements
- Pricing or packaging changes
- AI feature releases (particularly relevant given our AI-first positioning)
- Integration announcements (especially with CRM or project management tools)
- Positioning shifts in blog content or messaging

### 3. campaign-competitor-updates Slack Channel

Read the last 7 days of messages from the `campaign-competitor-updates` channel.

**Workflow:**
1. Use `slack_list_channels` to find the channel ID (search for "campaign-competitor-updates")
2. Use `slack_get_channel_messages` to read recent messages
3. Identify any companies or products mentioned
4. For each company found, do a brief web research profile:
   - What they do and their marketing product
   - Target market and positioning
   - Recent news, funding, or product launches
   - How they relate to your product (direct competitor, adjacent, emerging threat)

### 4. Internal Slack Discussions

Use `slack_search_public` to find competitive intelligence shared by the team in the past 7 days.

**Search terms:**
- "competitor", "competitive"
- Each competitor name: "mailchimp", "hubspot", "activecampaign", "brevo", "constant contact"
- "market", "alternative", "switching"

Focus on discussions in monday channels, not general Slack noise.

### 5. Previous Reports

Before writing the new report, read all existing reports in `your output folder/` (most recent first). Pay attention to:
- Themes and patterns flagged in prior weeks
- "Worth watching" items that may have developed
- Running comparison matrix for context
- Open questions from previous cycles

## Workflow

### Step 1 -- Read previous reports

Read the most recent 2-3 reports in `your output folder/` for running context. Note any open questions or "worth watching" items to follow up on.

### Step 2 -- Gather Reddit intelligence

Run web searches across the target subreddits. For each relevant post found:
- Note the subreddit and post title
- Capture the key insight (pain point, feature request, tool comparison, trend)
- Tag which competitors are mentioned
- Assess relevance to your product (high/medium/low)

Group findings by theme: feature gaps, pain points, tool comparisons, emerging trends.

### Step 3 -- Check competitor blogs and changelogs

Search for recent updates from each competitor. For each notable finding:
- What changed (feature, pricing, positioning)
- Source URL
- Significance for your product (high/medium/low)

### Step 4 -- Review campaign-competitor-updates channel

Read the Slack channel and profile any companies mentioned. For each:
- Company name
- What they do (brief, 1-2 sentences)
- Recent news or product updates
- Relevance to your product

### Step 5 -- Search internal Slack discussions

Find competitive intelligence shared by the team. Note anything that adds context to findings from other sources.

### Step 6 -- Synthesise and assess

Cross-reference all sources. Identify:
- **Converging signals** -- the same trend appearing in Reddit, blogs, and internal discussions
- **New threats** -- companies or features not previously on the radar
- **Opportunities** -- gaps competitors have that we could fill, or complaints we could address
- **Defensive needs** -- competitor moves that require a response

### Step 7 -- Produce outputs

Generate both the HTML section (for the morning briefing) and the full markdown report.

## Output

### HTML Output (Morning Briefing Section)

Generate HTML content using the routine template at `skills/system/routine-html-template.md`. Read that file for the full CSS and component patterns.

**If running as part of the start-of-day routine:** append this section to the morning briefing HTML (after the metrics section but before the footer). The `summarise-slack-and-suggest-tasks` skill assembles the initial file; this skill appends to it, like `review-daily-metrics` does.

**If running standalone:** save as a standalone HTML file to `your output folder/competitor-intel-YYYY-MM-DD.html` and open it.

#### HTML structure for this skill

```html
<hr class="divider">

<div class="section">
  <div class="section-title">Competitor Intel · Week of [date]</div>
</div>

<!-- This Week's Moves -->
<div class="section">
  <div class="section-title">This Week's Moves</div>

  <div class="card highlight">
    <div class="card-label highlight">[Competitor Name]</div>
    <div class="card-title">[What changed]</div>
    <div class="card-body">[Detail and significance]</div>
    <div class="card-meta">Source: [blog/changelog/press] · [date]</div>
  </div>

  <div class="card warning">
    <div class="card-label warning">[Competitor Name]</div>
    <div class="card-title">[Notable move worth watching]</div>
    <div class="card-body">[Detail]</div>
    <div class="card-meta">Source: [source] · [date]</div>
  </div>

  <div class="card urgent">
    <div class="card-label urgent">[Competitor Name]</div>
    <div class="card-title">[Significant competitive threat]</div>
    <div class="card-body">[Detail and recommended response]</div>
    <div class="card-meta">Source: [source] · [date]</div>
  </div>
</div>

<!-- Community Pulse (Reddit) -->
<div class="section">
  <div class="section-title">Community Pulse</div>

  <div class="card highlight">
    <div class="card-label highlight">[Theme: e.g. Feature Gaps]</div>
    <div class="card-title">[Key finding from Reddit]</div>
    <div class="card-body">[Summary of posts and sentiment. Mention subreddits.]</div>
  </div>

  <div class="card highlight">
    <div class="card-label highlight">[Theme: e.g. Tool Comparisons]</div>
    <div class="card-title">[Key finding]</div>
    <div class="card-body">[Summary]</div>
  </div>
</div>

<!-- Channel Intel -->
<div class="section">
  <div class="section-title">Channel Intel</div>

  <table>
    <thead>
      <tr><th>Company</th><th>What They Do</th><th>Recent News</th><th>Relevance</th></tr>
    </thead>
    <tbody>
      <tr>
        <td>[Company name]</td>
        <td>[Brief description]</td>
        <td>[Recent developments]</td>
        <td><span class="badge cyan">[Direct / Adjacent / Emerging]</span></td>
      </tr>
    </tbody>
  </table>

  <p class="quiet">Source: #campaign-competitor-updates · last 7 days</p>
</div>

<!-- Strategic Implications -->
<div class="section">
  <div class="section-title">Strategic Implications</div>

  <div class="card urgent">
    <div class="card-label urgent">Defensive</div>
    <div class="card-title">[What we need to respond to]</div>
    <div class="card-body">[Detail and urgency]</div>
  </div>

  <div class="card success">
    <div class="card-label success">Offensive</div>
    <div class="card-title">[Opportunity to gain advantage]</div>
    <div class="card-body">[Detail and recommendation]</div>
  </div>

  <div class="card warning">
    <div class="card-label warning">Worth watching</div>
    <div class="card-title">[Signal to monitor next week]</div>
    <div class="card-body">[Why it matters and what to look for]</div>
  </div>
</div>
```

Use card types based on significance:
- `card urgent` -- significant competitive threats requiring attention
- `card warning` -- emerging signals worth monitoring
- `card highlight` -- notable updates and intelligence
- `card success` -- opportunities for your product

If a section has no findings (e.g. no companies in the Slack channel), use: `<p class="quiet">No new companies flagged this week.</p>`

### Markdown Output (Full Report)

Save to `your output folder/competitor-analysis-YYYY-MM-DD.md` with this structure:

```markdown
# Weekly Competitor Analysis -- [Date]

## Executive Summary

[3-5 sentences. What happened this week that matters most. Connect to themes from previous weeks. Highlight any inflection points or significant shifts.]

## What Changed This Week

### [Competitor Name]
- [Change] -- [Source/evidence] -- [Significance: High/Medium/Low]

## Community Pulse (Reddit)

### [Theme]
- [Finding] -- [Subreddit] -- [Relevance to your product]

### [Theme]
- ...

## Channel Intel

### [Company Name]
- **What they do:** [Brief description]
- **Recent news:** [Developments]
- **Relevance:** [How they relate to your product]
- **Source:** #campaign-competitor-updates, [date]

## Internal Discussions

- [Key discussion or insight from Slack] -- [Channel] -- [Date]

## Patterns & Trends

- [Pattern] -- [How this connects to previous weeks' findings]
- [Emerging signal] -- [First flagged: date, or new this week]

## Positioning Impact

- **Strengths reinforced:** [Areas where our position improved or held]
- **Threats emerging:** [Competitor moves that could erode our position]
- **Opportunities opened:** [Gaps or shifts we could capitalise on]

## Strategic Implications

- **Defensive (act now):** [Moves to protect current position]
- **Offensive (plan for):** [Moves to gain advantage]
- **Worth watching:** [Signals to monitor next week]

## Open Questions

- [Questions to investigate in the next cycle]

---

*Previous report:* [link to previous week's file]
*Freshness:* Research conducted [date]
```

## Guidelines

- British English throughout
- No em dashes in HTML output (use "–" or rewrite)
- Lead with the most significant finding, not a chronological dump
- Be specific with numbers and sources: "3 Reddit threads in r/emailmarketing complained about Mailchimp's new pricing" not "users are unhappy"
- Use card urgency levels intentionally. Not every finding is urgent
- If a competitor had a quiet week, say so briefly rather than padding
- Cross-reference Reddit sentiment with what competitors actually shipped. User perception and reality often diverge
- Flag when a "worth watching" item from a previous report has escalated
- The HTML section should be scannable in 60 seconds. The full markdown report has the depth
- Keep the Channel Intel table focused. Only profile companies genuinely relevant to the competitive landscape
- Apply the `make-human-lite` skill before finalising any written output

## Competitor Set

Default set (adjustable by the user):

| Competitor | Category |
|------------|----------|
| Mailchimp | Direct -- SMB email marketing |
| HubSpot (Marketing Hub) | Direct -- marketing automation |
| ActiveCampaign | Direct -- email + automation |
| Brevo (formerly Sendinblue) | Direct -- SMB email + CRM |
| Constant Contact | Direct -- SMB email marketing |

Additional companies discovered in the `campaign-competitor-updates` channel are profiled ad hoc.

## Change Log

| Date | Change |
|------|--------|
| 18 Mar 2026 | Initial version. Combines Builders 2.0 analyzing-competitors methodology with Reddit, competitor blogs, and Slack channel intelligence. Outputs HTML (morning briefing section) and markdown (full report). |
