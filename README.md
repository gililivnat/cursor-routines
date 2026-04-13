# Cursor Routines

A system for running daily and weekly routines using Cursor AI agents. Built for product managers and anyone who wants their AI assistant to run structured morning briefings, end-of-day summaries, weekly reviews, and quarterly planning.

## What This Does

You trigger routines with **slash commands** — type `/goodmorning` to start your morning routine, `/goodnight` to wrap up the day, and so on. The system tracks progress with a state file, runs each step as a dedicated **skill** (a SKILL.md file the agent reads and follows), and avoids repeating work already done today.

The core ideas:
- **Opt-in via slash commands** — routines only run when you ask for them
- A **flag-based state machine** tracks what's been done today
- Each routine step is a **standalone skill** that can be run independently or improved over time
- Every routine produces a **visual HTML report** saved to disk and opened in the browser
- The agent resets the flags each morning and works through uncompleted steps

## Slash Commands

| Command | What it runs |
|---------|-------------|
| `/goodmorning` | Start of day (Slack DMs, product channels, summary + tasks, metrics, competitor intel on Mondays, Gmail, calendar + customer prep) |
| `/goodnight` | End of day (summarise the day, review meeting notes, Gmail highlights, task manager review) |
| `/planweek` | Week planning (priorities, tasks, calendar scan + customer prep offers) |
| `/weekreview` | Week summary + week planning (consolidate week's activity, metrics, meetings, email, then plan the next) |
| `/planquarter` | Quarterly planning (review quarter, draft next quarter priorities) |
| `/mondaytasks` | Monday recurring tasks (channel reviews, sync skills, check scheduled tasks) |
| `/checktasks` | Check for due or overdue scheduled tasks |
| `/voc` | Voice of the Customer report (multi-source customer insights) |
| `/biweekly` | Bi-weekly update prep (metrics snapshot, Slack scan, team prep message) |
| `/routines` | Show all available commands |

## How It Works

```
User types /goodmorning
  → Cursor rule detects the command
    → reads ORCHESTRATOR.md
      → reads state.md (date + flags)
        → if new day: reset flags
        → runs skills still marked false
          → each skill reads its SKILL.md, executes, marks itself done
          → skill produces an HTML report and opens it in the browser
        → if all done: tells you morning routine is complete
```

### The Flag Mechanism

The `state.md` file has YAML frontmatter tracking:

```yaml
date: "2026-03-13"           # last checked date
daily_start:
  flag: 0                     # 0 = not done, 1 = all steps complete
  review_slack_dms: false      # individual step tracking
  review_campaigns_channels: false
  ...
```

**Logic:**
1. If `date` < today → reset all daily flags to `false`, set `date` to today
2. If `date` == today and `flag` == 1 → skip (already done)
3. If `date` == today and `flag` == 0 → run steps still marked `false`
4. After each step completes → mark it `true`
5. When all steps are `true` → set the group `flag` to `1`

This means sessions can crash mid-routine and pick up exactly where they left off.

## Visual HTML Output

Every routine skill produces a **self-contained HTML file** — not just text in the chat. This is the primary output of the system.

### How it works

1. Skills gather data (from Slack, metrics, calendar, etc.)
2. They build an HTML file using a shared CSS template at `skills/system/routine-html-template.md`
3. The file is saved to your configured output folder (e.g. `your output folder/morning-briefing-2026-03-13.html`)
4. The file is opened in the default browser
5. A brief plain-text summary is also shown in the Cursor conversation

### The template system

The shared CSS template provides a dark, minimal design with semantic card types:

| Card type | Border colour | Use for |
|-----------|--------------|---------|
| `.card.urgent` | Red | Items needing immediate response |
| `.card.highlight` | Cyan | FYI items, notable updates |
| `.card.warning` | Amber | Things to watch, emerging signals |
| `.card.success` | Green | Completed items, positive trends |

It also provides components for metric grids, data tables, badges, person/channel tags, priority lists, and task rows.

### Interactive buttons

The HTML includes buttons like "Add to task manager", "Approve to send", and "Done". These are **visual intent markers, not executors**. Clicking a button toggles its visual state (e.g. changes "Add" to "Added") but does not call any API. The agent reads which buttons you toggled and performs the actual action (creating a task, sending a Slack message) when you return to Cursor and confirm.

This separation keeps the HTML files simple (no auth, no APIs, no server) while giving you a clean way to express intent.

### Output files

| Routine | Filename pattern |
|---------|-----------------|
| Morning briefing | `morning-briefing-YYYY-MM-DD.html` |
| End of day | `eod-summary-YYYY-MM-DD.html` |
| Weekly summary | `week-summary-YYYY-MM-DD.html` |
| Weekly plan | `week-plan-YYYY-MM-DD.html` |
| Quarterly plan | `quarter-plan-QN-YYYY.html` |

## Routines Included

### Daily: Start of Day (`/goodmorning`)
1. **Review Slack DMs** — scan personal messages, flag what needs a response
2. **Review product channels** — read all Slack channels matching a keyword, summarise activity
3. **Summarise + suggest tasks** — consolidate Slack findings, suggest actionable tasks
4. **Review daily metrics** — query your data warehouse, surface highlights and anomalies
5. **Weekly competitor intel** (Monday only) — scan competitor channels and sources for updates
6. **Review Gmail** — scan inbox for urgent/important emails, highlight what needs attention
7. **Review upcoming meetings** — scan today's calendar, prepare context for each meeting, detect customer meetings and offer to run a customer interview prep

### Daily: End of Day (`/goodnight`)
1. **Summarise the day** — review product Slack messages, highlight decisions and open items
2. **Review meeting notes** — check AI notetaker for today's meetings, extract action items
3. **Review Gmail (EOD)** — highlight unanswered emails and what was handled during the day
4. **Review task manager** — show today's tasks with completion status, offer to postpone incomplete items

### Weekly
- **Plan the week** (`/planweek`, Monday) — review carry-overs, scan the full week's calendar (with customer meeting detection), set top 3 priorities
- **Summarise the week** (`/weekreview`, Friday) — consolidate activity, metrics, meetings, email highlights, then automatically run week planning for the following week
- **Plan the quarter** (`/planquarter`, last month of Q) — review OKRs, draft next quarter priorities

### Monday Recurring Tasks (`/mondaytasks`)
- Review support channel (7-day lookback)
- Review churn channel (7-day lookback, with automated follow-up email workflow)
- Sync external skills repos
- Check scheduled tasks for anything due

### Voice of the Customer (`/voc`)
Aggregates customer insights from multiple sources (usage data, support channels, churn signals, forums, analytics, internal discussions) into a themed VoC report.

### Bi-weekly Update Prep (`/biweekly`)
Prepares for director-level updates: metrics snapshot, Slack activity scan, team prep message.

### Customer Meeting Detection

The calendar skills (both daily and weekly) automatically detect **customer/external meetings** — any meeting where at least one attendee has a non-internal email domain. When a customer meeting is found:

1. It's flagged prominently in the output (urgent card style)
2. The external attendee's company is identified
3. The agent offers to run a **customer interview prep** skill, which pulls:
   - Product usage data from your data warehouse
   - Financial/billing history
   - Internal feedback from Slack channels
   - Company website research
   - Suggested interview questions grounded in the data

This means your morning briefing or weekly plan automatically surfaces "you have a call with Acme Corp at 2pm — want me to pull their data?" without you needing to ask.

## Structure

```
cursor-routines/
├── README.md
├── config.example.yaml        — configuration template (MCP names, paths, product settings)
├── cursor-rule-example.md     — example Cursor rule to activate the system
├── routines/
│   ├── ORCHESTRATOR.md        — slash commands, flag logic, execution rules
│   ├── state.example.md       — template state file (copy to state.md)
│   ├── daily/
│   │   ├── start-of-day.md
│   │   └── end-of-day.md
│   ├── weekly/
│   │   ├── week-summary.md
│   │   ├── week-planning.md
│   │   └── quarterly-planning.md
│   └── biweekly/
│       └── biweekly-update.md
└── skills/
    ├── review-slack-dms/          — morning DM scan
    ├── review-campaigns-channels/ — product channel review
    ├── summarise-slack-and-suggest-tasks/ — consolidated summary + task suggestions
    ├── review-daily-metrics/      — data warehouse metrics check
    ├── weekly-competitor-intel/   — Monday competitor scan
    ├── review-upcoming-meetings/  — calendar review + customer detection
    ├── review-gmail/              — email triage
    ├── summarise-campaigns-day/   — end-of-day Slack summary
    ├── review-notetaker-meetings/ — meeting notes extraction
    ├── review-todoist-tasks/      — task manager review (EOD)
    ├── summarise-week/            — weekly activity consolidation
    ├── plan-week/                 — weekly priority setting
    ├── plan-quarter/              — quarterly planning
    ├── prepare-biweekly-update/   — director update prep
    ├── prepare-customer-interview/ — customer meeting prep dossier
    ├── create-presentation/       — default deck workflow (any presentation the user asks for)
    ├── voice-of-the-customer/     — multi-source VoC report
    ├── onboarding/                — guided first-time setup
    └── system/
        ├── routine-html-template.md     — shared CSS/HTML template for routine output
        └── presentation-html-template.md — template for standalone presentations
```

## Setup

### Quick Setup Checklist

1. Copy `config.example.yaml` to `config.yaml` and fill in your values
2. Copy `routines/state.example.md` to `routines/state.md`
3. Install the Cursor rule from `cursor-rule-example.md` to `.cursor/rules/routines.mdc`
4. Connect your MCP servers in Cursor (names must match what you set in `config.yaml`)
5. Install skills to `~/.cursor/skills/` (see below)
6. Run the **onboarding skill** — it reads `config.yaml` and updates all placeholder values across skill files

### 1. Copy the files into your workspace

Place the `routines/` and `skills/` folders wherever makes sense in your project structure. Update the file paths in the orchestrator and routine files accordingly.

### 2. Configure with config.yaml

```bash
cp config.example.yaml config.yaml
```

Open `config.yaml` and fill in:
- **workspace_path** — absolute path to your workspace root
- **product_name** — your product or project name
- **channel_keyword** — keyword to find your product's Slack channels (e.g. "marketing", "product")
- **slack_user_id** — your Slack member ID (starts with U)
- **MCP server names** — the names you gave your MCP servers in Cursor
- **output_path** — where HTML reports are saved

### 3. Create your state file

```bash
cp routines/state.example.md routines/state.md
```

Edit the date to match today.

### 4. Install the Cursor rule

Copy `cursor-rule-example.md` to `.cursor/rules/routines.mdc` (or whatever naming convention you use). This tells Cursor to listen for slash commands and trigger the right routine.

### 5. Install skills to Cursor

For Cursor to auto-discover skills, copy each SKILL.md to `~/.cursor/skills/[skill-name]/SKILL.md`:

```bash
for skill in skills/*/; do
  name=$(basename "$skill")
  mkdir -p ~/.cursor/skills/$name
  cp "$skill/SKILL.md" ~/.cursor/skills/$name/SKILL.md
done
```

### 6. Connect MCP servers

> **Important:** MCP server names in skill files must match your Cursor MCP configuration exactly. If you used `config.yaml` + the onboarding skill, this is handled automatically. If setting up manually, search skill files for the placeholder names listed below and replace them with your actual MCP server names.

The skills need these Cursor MCP connections:

| MCP | Used by | Level |
|-----|---------|-------|
| **Slack** | DM review, channel review, summarise day/week, task suggestions | Core — most skills depend on this |
| **Data warehouse** (e.g. your data warehouse, BigQuery) | Daily metrics, customer prep, VoC | Core — metrics and data skills |
| **Google Workspace** (Calendar + Gmail) | Calendar review, Gmail review, customer meeting detection | Core — calendar and email skills |
| **Meeting notetaker** | Meeting notes review | Core — meeting review skill |
| **Task manager** (e.g. task manager, Linear) | Task suggestions, EOD task review, HTML action buttons, weekly planning | Enrichment (see note below) |
| **Analytics platform** (e.g. analytics platform, Amplitude) | VoC report, customer interview prep | Optional — only used by `/voc` and customer prep |

#### A note on task manager integration

The task manager is labelled "enrichment" rather than "optional" because it is referenced across many skills, not just one. It appears in:
- Morning summary (task suggestions with "Add to task manager" buttons)
- End of day (task review with completion status, postpone buttons)
- HTML template (button CSS and interaction patterns)
- Weekly planning (carry-over task detection)

Without a task manager MCP connected, these features are skipped gracefully — the system still works. But if you want to fully remove task manager references rather than just skip them, you'll need to search skill files for "task manager" and update the HTML template buttons.

### Manual Setup (Without the Onboarding Skill)

If you prefer to configure manually instead of using the onboarding skill, here are the placeholder values that appear across skill files. Search and replace each one:

| Placeholder | What to set it to | Where it appears |
|-------------|-------------------|-----------------|
| `YOUR_SLACK_USER_ID` | Your Slack member ID (e.g. `U01ABC2DEF3`) | `review-slack-dms`, `summarise-slack-and-suggest-tasks` |
| `CHANNEL_KEYWORD` | Keyword to match your Slack channels | `review-campaigns-channels`, `summarise-campaigns-day` |
| `your output folder/` | Path where HTML reports are saved | Multiple skills (output file paths) |
| `your scheduled tasks file` | Path to your recurring tasks list | `plan-week`, orchestrator |
| `your data guide` | Path to data schema/metric documentation | `review-daily-metrics`, `plan-quarter` |
| `your Slack MCP` | Your Slack MCP server name in Cursor | Multiple skills |
| `your data warehouse MCP` | Your data warehouse MCP server name | `review-daily-metrics`, `plan-quarter`, `voice-of-the-customer` |
| `your Google Workspace MCP` | Your Google Workspace MCP server name | `review-upcoming-meetings`, `review-gmail`, `plan-week` |
| `your meeting notetaker MCP` | Your notetaker MCP server name | `review-notetaker-meetings` |
| `your analytics MCP` | Your analytics MCP server name | `voice-of-the-customer`, `prepare-customer-interview` |

## Customising

### Adding a new routine step

1. Create a new skill: `skills/[name]/SKILL.md`
2. Add a field for it in `state.md` under the relevant group
3. Add a step entry in the relevant routine file (`daily/start-of-day.md`, etc.)
4. Add it to the skills registry in `ORCHESTRATOR.md`

### Adding a new slash command

1. Define the routine files and skills as above
2. Add the command to the slash commands table in `ORCHESTRATOR.md`
3. Add the command to the Cursor rule (`cursor-rule-example.md`)
4. Add it to the `/routines` display list

### Removing a routine step

1. Comment out or remove the field from `state.md`
2. Remove the step from the routine file
3. Optionally delete the skill folder

### Changing cadence

Edit the routine files and the orchestrator's execution logic.

### Customising the HTML template

Edit `skills/system/routine-html-template.md` to change colours, fonts, spacing, or add new component patterns. All skills reference this single template, so changes apply everywhere.

## Design Principles

- **Opt-in, not automatic:** routines only run when you ask for them via slash commands. No surprise interruptions.
- **Skill-per-step:** every routine step is a standalone skill that knows exactly what to do. Skills can be improved independently and reused outside the routine system.
- **Visual output:** every routine produces an HTML report you can browse, scroll, and revisit — not just chat text that scrolls away.
- **State as YAML:** simple, human-readable, easy to debug. No database needed.
- **Idempotent:** if a session crashes mid-routine, the next session picks up where it left off (incomplete steps stay `false`).
- **Progressive:** placeholder skills let you define the full routine now and activate steps as you connect new tools.
- **Context-aware:** calendar and email skills detect what matters (customer meetings, urgent emails) and proactively offer deeper preparation.
- **Intent, not execution:** HTML buttons express what you want to do (add a task, send a message). The agent handles execution after you confirm.

## Requirements

- [Cursor IDE](https://cursor.com/) with agent mode
- Slack MCP (for most skills)
- A data warehouse MCP (for metrics)
- Google Workspace MCP (for Gmail and Calendar)
- A meeting notetaker MCP (for meeting review)
- A task manager MCP (for task suggestions and review — enrichment, not strictly required)

## Licence

MIT
