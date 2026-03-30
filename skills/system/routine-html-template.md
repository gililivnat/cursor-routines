# Routine HTML Output Template

All routine skills output their results as self-contained HTML files, saved to `your output folder/` and opened in the browser for the user.

## File Naming

Each routine output is saved with a predictable name:
- Daily start: `your output folder/morning-briefing-YYYY-MM-DD.html`
- Daily end: `your output folder/eod-summary-YYYY-MM-DD.html`
- Weekly summary: `your output folder/week-summary-YYYY-MM-DD.html`
- Weekly plan: `your output folder/week-plan-YYYY-MM-DD.html`
- Weekly product summary: `your output folder/product-summary-YYYY-MM-DD.html`
- Quarterly plan: `your output folder/quarter-plan-QN-YYYY.html`

## How to Use

1. After gathering all data for a routine, build the HTML string using the base styles and component patterns below
2. Save the file to `your output folder/` using the naming convention above
3. Open the file with `open` (macOS) so it launches in the default browser
4. Also present a brief plain-text summary in the conversation so the user has context without switching windows

## Opening the File

```bash
open [your-workspace-path] user-Workspace/your output folder/[filename].html
```

## Base HTML Structure

Every routine HTML file follows this structure. Copy the full `<style>` block as-is into every output file.

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>{{TITLE}}</title>
<style>
  * { margin: 0; padding: 0; box-sizing: border-box; }
  body {
    background: #0a0a0a;
    color: #d4d4d4;
    font-family: -apple-system, BlinkMacSystemFont, 'Inter', 'Segoe UI', sans-serif;
    font-size: 15px;
    line-height: 1.6;
    padding: 40px 20px;
    max-width: 860px;
    margin: 0 auto;
  }

  /* Header */
  .header {
    border-bottom: 1px solid #222;
    padding-bottom: 20px;
    margin-bottom: 32px;
  }
  .header h1 {
    font-size: 22px;
    font-weight: 600;
    color: #ffffff;
    letter-spacing: -0.3px;
  }
  .header .subtitle {
    font-size: 13px;
    color: #666;
    margin-top: 4px;
  }

  /* Sections */
  .section {
    margin-bottom: 32px;
  }
  .section-title {
    font-size: 13px;
    font-weight: 600;
    color: #666;
    text-transform: uppercase;
    letter-spacing: 0.8px;
    margin-bottom: 12px;
  }

  /* Cards */
  .card {
    background: #141414;
    border: 1px solid #1e1e1e;
    border-radius: 8px;
    padding: 16px 20px;
    margin-bottom: 8px;
  }
  .card.urgent {
    border-left: 3px solid #f87171;
  }
  .card.highlight {
    border-left: 3px solid #00d4ff;
  }
  .card.success {
    border-left: 3px solid #4ade80;
  }
  .card.warning {
    border-left: 3px solid #fbbf24;
  }

  /* Card content */
  .card-label {
    font-size: 11px;
    font-weight: 600;
    text-transform: uppercase;
    letter-spacing: 0.6px;
    margin-bottom: 4px;
  }
  .card-label.urgent { color: #f87171; }
  .card-label.highlight { color: #00d4ff; }
  .card-label.success { color: #4ade80; }
  .card-label.warning { color: #fbbf24; }
  .card-title {
    font-size: 15px;
    font-weight: 500;
    color: #e5e5e5;
  }
  .card-body {
    font-size: 14px;
    color: #999;
    margin-top: 4px;
  }
  .card-meta {
    font-size: 12px;
    color: #555;
    margin-top: 6px;
  }

  /* Channel tag */
  .channel {
    display: inline-block;
    background: #1a1a2e;
    color: #00d4ff;
    font-size: 12px;
    padding: 2px 8px;
    border-radius: 4px;
    font-weight: 500;
  }

  /* Person tag */
  .person {
    color: #c084fc;
    font-weight: 500;
  }

  /* Metric row */
  .metric-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(180px, 1fr));
    gap: 10px;
    margin-bottom: 8px;
  }
  .metric-card {
    background: #141414;
    border: 1px solid #1e1e1e;
    border-radius: 8px;
    padding: 16px;
    text-align: center;
  }
  .metric-value {
    font-size: 28px;
    font-weight: 600;
    color: #ffffff;
    line-height: 1.2;
  }
  .metric-label {
    font-size: 12px;
    color: #666;
    margin-top: 4px;
  }
  .metric-change {
    font-size: 13px;
    margin-top: 6px;
    font-weight: 500;
  }
  .metric-change.up { color: #4ade80; }
  .metric-change.down { color: #f87171; }
  .metric-change.flat { color: #666; }

  /* Table */
  table {
    width: 100%;
    border-collapse: collapse;
    font-size: 14px;
  }
  th {
    text-align: left;
    font-size: 12px;
    font-weight: 600;
    color: #555;
    text-transform: uppercase;
    letter-spacing: 0.5px;
    padding: 8px 12px;
    border-bottom: 1px solid #222;
  }
  td {
    padding: 10px 12px;
    border-bottom: 1px solid #1a1a1a;
    color: #d4d4d4;
  }
  tr:hover td {
    background: #141414;
  }

  /* Tag / badge */
  .badge {
    display: inline-block;
    font-size: 11px;
    font-weight: 600;
    padding: 2px 8px;
    border-radius: 4px;
    text-transform: uppercase;
    letter-spacing: 0.4px;
  }
  .badge.green { background: #052e16; color: #4ade80; }
  .badge.red { background: #2a0a0a; color: #f87171; }
  .badge.amber { background: #2a1f00; color: #fbbf24; }
  .badge.cyan { background: #0a1a2e; color: #00d4ff; }
  .badge.gray { background: #1a1a1a; color: #888; }

  /* task manager button */
  .btn-todoist {
    display: inline-flex;
    align-items: center;
    gap: 6px;
    background: transparent;
    border: 1px solid #333;
    color: #999;
    font-size: 12px;
    padding: 4px 12px;
    border-radius: 6px;
    cursor: pointer;
    transition: all 0.15s ease;
    font-family: inherit;
  }
  .btn-todoist:hover {
    border-color: #00d4ff;
    color: #00d4ff;
    background: #0a1a2e;
  }
  .btn-todoist.added {
    border-color: #4ade80;
    color: #4ade80;
    background: #052e16;
  }
  .btn-todoist .icon { font-size: 14px; }

  /* Approve for sending button (visual toggle only, agent sends via MCP) */
  .btn-slack {
    display: inline-flex;
    align-items: center;
    gap: 8px;
    background: transparent;
    border: 1px solid #444;
    color: #ccc;
    font-size: 13px;
    font-weight: 500;
    padding: 8px 20px;
    border-radius: 8px;
    cursor: pointer;
    transition: all 0.15s ease;
    font-family: inherit;
    margin-top: 12px;
  }
  .btn-slack:hover {
    border-color: #4ade80;
    color: #4ade80;
    background: #0a2e16;
  }
  .btn-slack.sent {
    border-color: #4ade80;
    color: #4ade80;
    background: #052e16;
  }
  .btn-slack .slack-icon { font-size: 16px; }

  /* Instruction bar */
  .send-instructions {
    background: #1a1a2e;
    border: 1px solid #252545;
    border-radius: 8px;
    padding: 12px 16px;
    font-size: 13px;
    color: #8888cc;
    margin-bottom: 24px;
    line-height: 1.5;
  }
  .send-instructions strong { color: #00d4ff; }

  /* Message preview box */
  .message-preview {
    background: #111;
    border: 1px solid #222;
    border-radius: 8px;
    padding: 20px;
    margin-bottom: 8px;
    font-size: 14px;
    line-height: 1.7;
    color: #c0c0c0;
    white-space: pre-wrap;
  }
  .message-preview .msg-bold {
    color: #e5e5e5;
    font-weight: 600;
  }
  .message-header {
    display: flex;
    align-items: center;
    justify-content: space-between;
    margin-bottom: 8px;
  }
  .message-channel {
    display: inline-flex;
    align-items: center;
    gap: 6px;
    font-size: 13px;
    color: #888;
  }
  .message-channel .channel { font-size: 13px; }

  /* Task item with button */
  .task-row {
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: 12px;
    padding: 10px 0;
    border-bottom: 1px solid #1a1a1a;
    transition: opacity 0.2s ease;
  }
  .task-row:last-child { border-bottom: none; }
  .task-row.completed { opacity: 0.4; }
  .task-row.completed .task-content { text-decoration: line-through; }
  .task-content {
    flex: 1;
    color: #d4d4d4;
    font-size: 14px;
  }
  .task-actions {
    display: flex;
    gap: 6px;
    flex-shrink: 0;
  }
  .task-priority {
    font-size: 11px;
    font-weight: 600;
    text-transform: uppercase;
    letter-spacing: 0.4px;
    flex-shrink: 0;
  }

  /* Done button */
  .btn-done {
    display: inline-flex;
    align-items: center;
    gap: 6px;
    background: transparent;
    border: 1px solid #333;
    color: #999;
    font-size: 12px;
    padding: 4px 12px;
    border-radius: 6px;
    cursor: pointer;
    transition: all 0.15s ease;
    font-family: inherit;
  }
  .btn-done:hover {
    border-color: #4ade80;
    color: #4ade80;
    background: #052e16;
  }
  .btn-done.done {
    border-color: #4ade80;
    color: #4ade80;
    background: #052e16;
  }
  .btn-done .icon { font-size: 14px; }

  /* Postpone button */
  .btn-postpone {
    display: inline-flex;
    align-items: center;
    gap: 6px;
    background: transparent;
    border: 1px solid #333;
    color: #999;
    font-size: 12px;
    padding: 4px 12px;
    border-radius: 6px;
    cursor: pointer;
    transition: all 0.15s ease;
    font-family: inherit;
  }
  .btn-postpone:hover {
    border-color: #fbbf24;
    color: #fbbf24;
    background: #2a1f00;
  }
  .btn-postpone.postponed {
    border-color: #fbbf24;
    color: #fbbf24;
    background: #2a1f00;
  }
  .btn-postpone .icon { font-size: 14px; }

  /* Open in your task manager link-button */
  .btn-open-todoist {
    display: inline-flex;
    align-items: center;
    gap: 6px;
    background: transparent;
    border: 1px solid #333;
    color: #999;
    font-size: 12px;
    padding: 4px 12px;
    border-radius: 6px;
    cursor: pointer;
    transition: all 0.15s ease;
    font-family: inherit;
    text-decoration: none;
  }
  .btn-open-todoist:hover {
    border-color: #00d4ff;
    color: #00d4ff;
    background: #0a1a2e;
  }
  .btn-open-todoist .icon { font-size: 14px; }

  /* List items */
  .item-list {
    list-style: none;
    padding: 0;
  }
  .item-list li {
    padding: 8px 0;
    border-bottom: 1px solid #1a1a1a;
    font-size: 14px;
  }
  .item-list li:last-child { border-bottom: none; }

  /* Quiet / muted */
  .quiet { color: #555; font-size: 13px; }

  /* Divider */
  .divider {
    border: none;
    border-top: 1px solid #1e1e1e;
    margin: 24px 0;
  }

  /* Footer */
  .footer {
    margin-top: 40px;
    padding-top: 16px;
    border-top: 1px solid #1e1e1e;
    font-size: 12px;
    color: #444;
  }

  /* Priority list (numbered) */
  .priority-list {
    list-style: none;
    padding: 0;
    counter-reset: priority;
  }
  .priority-list li {
    counter-increment: priority;
    padding: 12px 0 12px 36px;
    border-bottom: 1px solid #1a1a1a;
    position: relative;
  }
  .priority-list li:last-child { border-bottom: none; }
  .priority-list li::before {
    content: counter(priority);
    position: absolute;
    left: 0;
    top: 12px;
    width: 24px;
    height: 24px;
    background: #1a1a2e;
    color: #00d4ff;
    border-radius: 50%;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 12px;
    font-weight: 600;
  }

  /* Summary paragraph */
  .summary-text {
    font-size: 15px;
    line-height: 1.7;
    color: #b0b0b0;
    max-width: 720px;
  }

  /* Responsive */
  @media (max-width: 600px) {
    body { padding: 20px 16px; }
    .metric-grid { grid-template-columns: repeat(2, 1fr); }
  }
</style>
</head>
<body>
  {{CONTENT}}
  <div class="footer">Generated by your product ops · {{DATE}}</div>
</body>
</html>
```

## Component Reference

Below are the HTML patterns for common content types. Combine them as needed.

### Header

Each routine type uses a personalised greeting with an emoji:

| Routine | Headline |
|---------|----------|
| Morning briefing | `☀️ Good morning, the user` |
| End of day | `🌙 Good evening, the user` |
| Weekly plan (Monday) | `📋 Let's plan the week, the user` |
| Weekly summary (Friday) | `📊 Let's review the week, the user` |
| Quarterly plan | `🗺️ Let's plan the quarter, the user` |

```html
<div class="header">
  <h1>☀️ Good morning, the user</h1>
  <div class="subtitle">Friday 14 March 2026 · 8 channels checked · 12 DMs reviewed</div>
</div>
```

### Section title

```html
<div class="section">
  <div class="section-title">Needs Response</div>
  <!-- cards go here -->
</div>
```

### Cards (with urgency levels)

```html
<div class="card urgent">
  <div class="card-label urgent">Needs reply</div>
  <div class="card-title"><span class="person">Shannon Klett</span> asked about unified editor timeline</div>
  <div class="card-body">Wants confirmation on the compliance warning copy before release.</div>
  <div class="card-meta"><span class="channel">#campaigns-builders</span> · 2 hours ago</div>
</div>

<div class="card highlight">
  <div class="card-label highlight">FYI</div>
  <div class="card-title">1M board alpha update from Oded</div>
  <div class="card-body">Resound imported 3M addresses, testing expected this week.</div>
  <div class="card-meta"><span class="channel">#campaigns-1m</span> · yesterday</div>
</div>
```

### Metrics grid

```html
<div class="metric-grid">
  <div class="metric-card">
    <div class="metric-value">235</div>
    <div class="metric-label">Product sent/day</div>
    <div class="metric-change up">+17% vs last week</div>
  </div>
  <div class="metric-card">
    <div class="metric-value">769</div>
    <div class="metric-label">Paying accounts</div>
    <div class="metric-change up">+13 this week</div>
  </div>
  <div class="metric-card">
    <div class="metric-value">97%</div>
    <div class="metric-label">Delivery rate</div>
    <div class="metric-change flat">Stable</div>
  </div>
</div>
```

### Table

```html
<table>
  <thead>
    <tr><th>Metric</th><th>This week</th><th>Last week</th><th>Change</th></tr>
  </thead>
  <tbody>
    <tr>
      <td>Product sent/day</td>
      <td>235</td>
      <td>201</td>
      <td><span class="badge green">+17%</span></td>
    </tr>
    <tr>
      <td>New installs</td>
      <td>15</td>
      <td>22</td>
      <td><span class="badge red">-32%</span></td>
    </tr>
  </tbody>
</table>
```

### Suggested tasks (with task manager and Done buttons)

Each task row has two buttons: **Done** (marks the task as completed, strikes through and fades the row) and **Add to your task manager** (marks the task for task manager creation). Both are visual toggles. The agent handles actual task manager creation based on the user's confirmation.

```html
<div class="section">
  <div class="section-title">Suggested Tasks</div>
  <div class="task-row">
    <div class="task-content">Reply to Shannon about unified editor compliance copy</div>
    <div class="task-actions">
      <button class="btn-done" onclick="this.classList.toggle('done'); this.closest('.task-row').classList.toggle('completed'); this.innerHTML = this.classList.contains('done') ? '<span class=\'icon\'>✓</span> Done' : '<span class=\'icon\'>○</span> Done'">
        <span class="icon">○</span> Done
      </button>
      <button class="btn-todoist" onclick="this.classList.toggle('added'); this.innerHTML = this.classList.contains('added') ? '<span class=\'icon\'>✓</span> Added' : '<span class=\'icon\'>+</span> Add to your task manager'">
        <span class="icon">+</span> Add to your task manager
      </button>
    </div>
  </div>
  <div class="task-row">
    <div class="task-content">Review Qodo setup and agree on usage with team</div>
    <div class="task-actions">
      <button class="btn-done" onclick="this.classList.toggle('done'); this.closest('.task-row').classList.toggle('completed'); this.innerHTML = this.classList.contains('done') ? '<span class=\'icon\'>✓</span> Done' : '<span class=\'icon\'>○</span> Done'">
        <span class="icon">○</span> Done
      </button>
      <button class="btn-todoist" onclick="this.classList.toggle('added'); this.innerHTML = this.classList.contains('added') ? '<span class=\'icon\'>✓</span> Added' : '<span class=\'icon\'>+</span> Add to your task manager'">
        <span class="icon">+</span> Add to your task manager
      </button>
    </div>
  </div>
</div>
```

### Priority list (numbered, for top priorities)

```html
<ol class="priority-list">
  <li>
    <div class="card-title">Ship workflow triggers</div>
    <div class="card-body">Nick confirmed staging is clear. Aim for release today.</div>
  </li>
  <li>
    <div class="card-title">Review shared contacts migration deck</div>
    <div class="card-body">Chandra's hack is in, but needs Shannon's sign-off before broader rollout.</div>
  </li>
</ol>
```

### Badges

Use for status, trend direction, and labels:

```html
<span class="badge green">Shipped</span>
<span class="badge cyan">In progress</span>
<span class="badge amber">Watch</span>
<span class="badge red">Blocked</span>
<span class="badge gray">Quiet</span>
```

### Slack message preview with approval button

Used for draft Slack messages (weekly summaries, announcements). The buttons are **approval toggles only**. They do NOT send anything to Slack. The agent handles actual sending via Slack MCP when the user confirms in the Cursor conversation.

Always include the instruction bar above the message sections:

```html
<div class="send-instructions">
  <strong>How sending works:</strong> These buttons mark messages as approved. To actually send, go back to Cursor and tell the agent which messages to post. The agent will use Slack to send them on your behalf.
</div>

<div class="section">
  <div class="message-header">
    <div class="section-title">Team Update</div>
    <div class="message-channel">Will post to <span class="channel">#monday-campaigns</span></div>
  </div>
  <div class="message-preview">
    <span class="msg-bold">Weekly Update | 14 March 2026</span>

    Good week. Here's what happened:
    • Sidekick AI is live for all accounts
    • Campaign sends up 17% vs last week
    • Workflow triggers deploying today

    Numbers:
    • 235 campaigns sent/day (up from 201)
    • 769 paying accounts (+13 this week)

    Big shoutout to Uras, Noam, and Nick.
    Have a good weekend.
  </div>
  <button class="btn-slack" onclick="this.classList.toggle('sent'); this.innerHTML = this.classList.contains('sent') ? '<span class=\'slack-icon\'>✓</span> Approved' : '<span class=\'slack-icon\'>☐</span> Approve to send'">
    <span class="slack-icon">☐</span> Approve to send
  </button>
</div>
```

### Quiet / empty state

```html
<p class="quiet">No messages in the last 24 hours.</p>
```

---

## Design Principles

- **Black, minimal.** Dark backgrounds, thin borders, generous spacing.
- **Cyan (#00d4ff) is the primary accent.** Use for highlights, active badges, links, and interactive elements.
- **Green for positive.** Metrics going up, shipped items, completed tasks.
- **Amber for caution.** Things to watch, emerging signals.
- **Red for urgent.** Items needing immediate response, negative trends.
- **Purple for people.** Person names use purple (#c084fc) so they stand out.
- **Data first.** Lead with numbers and metrics, then context.
- **No decoration for decoration's sake.** Every visual element carries meaning.
