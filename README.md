# Cursor Routines

A system for running daily and weekly routines using Cursor AI agents. Built for product managers and anyone who wants their AI assistant to run structured morning briefings, end-of-day summaries, weekly reviews, and quarterly planning.

## What This Does

You trigger routines with **slash commands** — type `/goodmorning` to start your morning routine, `/goodnight` to wrap up the day, and so on. The system tracks progress with a state file, runs each step as a dedicated **skill** (a SKILL.md file the agent reads and follows), and avoids repeating work already done today.

The core idea is simple:
- **Opt-in via slash commands** — routines only run when you ask for them
- A **flag-based state machine** tracks what's been done today
- Each routine step is a **standalone skill** that can be run independently or improved over time
- The agent resets the flags each morning and works through uncompleted steps

## Slash Commands

| Command | What it runs |
|---------|-------------|
| `/goodmorning` | Start of day (Slack DMs, product channels, summary + tasks, metrics, Gmail, calendar + customer prep) |
| `/goodnight` | End of day (summarise the day, review meeting notes, Gmail highlights) |
| `/planweek` | Week planning (priorities, tasks, calendar scan + customer prep offers) |
| `/weekreview` | Week summary (consolidate week's activity, metrics, meetings, email) |
| `/planquarter` | Quarterly planning (review quarter, draft next quarter priorities) |
| `/checktasks` | Check for due or overdue scheduled tasks |
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

## Routines Included

### Daily: Start of Day (`/goodmorning`)
1. **Review Slack DMs** — scan personal messages, flag what needs a response
2. **Review product channels** — read all Slack channels matching a keyword, summarise activity
3. **Summarise + suggest tasks** — consolidate Slack findings, suggest actionable tasks
4. **Review daily metrics** — query your data warehouse, surface highlights and anomalies
5. **Review Gmail** — scan inbox for urgent/important emails, highlight what needs attention
6. **Review upcoming meetings** — scan today's calendar, prepare context for each meeting, detect customer meetings and offer to run a customer interview prep

### Daily: End of Day (`/goodnight`)
1. **Summarise the day** — review product Slack messages, highlight decisions and open items
2. **Review meeting notes** — check AI notetaker for today's meetings, extract action items
3. **Review Gmail (EOD)** — highlight unanswered emails and what was handled during the day

### Weekly
- **Plan the week** (`/planweek`, Monday) — review carry-overs, scan the full week's calendar (with customer meeting detection), set top 3 priorities
- **Summarise the week** (`/weekreview`, Friday) — consolidate activity, metrics, meetings, email highlights
- **Plan the quarter** (`/planquarter`, last month of Q) — review OKRs, draft next quarter priorities

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
├── cursor-rule-example.md     — example Cursor rule to activate the system
├── routines/
│   ├── ORCHESTRATOR.md        — slash commands, flag logic, execution rules
│   ├── state.example.md       — template state file (copy to state.md)
│   ├── daily/
│   │   ├── start-of-day.md
│   │   └── end-of-day.md
│   └── weekly/
│       ├── week-summary.md
│       ├── week-planning.md
│       └── quarterly-planning.md
└── skills/                    — one SKILL.md per routine step
    ├── review-slack-dms/
    ├── review-campaigns-channels/
    ├── summarise-slack-and-suggest-tasks/
    ├── review-daily-metrics/
    ├── review-upcoming-meetings/
    ├── review-gmail/
    ├── summarise-campaigns-day/
    ├── review-notetaker-meetings/
    ├── summarise-week/
    ├── plan-week/
    └── plan-quarter/
```

## Setup

### 1. Copy the files into your workspace

Place the `routines/` and `skills/` folders wherever makes sense in your project structure. Update the file paths in the orchestrator and routine files accordingly.

### 2. Create your state file

```bash
cp routines/state.example.md routines/state.md
```

Edit the date and fields to match today and your active skills.

### 3. Install the Cursor rule

Copy `cursor-rule-example.md` to `.cursor/rules/routines.mdc` (or whatever naming convention you use). This tells Cursor to listen for slash commands and trigger the right routine.

### 4. Install skills to Cursor

For Cursor to auto-discover skills, copy each SKILL.md to `~/.cursor/skills/[skill-name]/SKILL.md`:

```bash
for skill in skills/*/; do
  name=$(basename "$skill")
  mkdir -p ~/.cursor/skills/$name
  cp "$skill/SKILL.md" ~/.cursor/skills/$name/SKILL.md
done
```

### 5. Configure for your product

- In `review-campaigns-channels` and `summarise-campaigns-day`: change `CHANNEL_KEYWORD` to match your product's Slack channels
- In `review-slack-dms`: set `YOUR_SLACK_USER_ID`
- In `review-daily-metrics`: customise the Metrics to Track table for your product
- In `review-notetaker-meetings`: adapt tool names to your AI notetaker MCP
- In `review-upcoming-meetings`: adjust the list of internal email domains (defaults to your company domain) and common tool domains to exclude from customer detection
- In `review-gmail`: adjust priority rules (which senders are urgent, which are FYI)

### 6. Connect required MCPs

The skills need these Cursor MCP connections:

| MCP | Used by | Required? |
|-----|---------|-----------|
| **Slack MCP** | DM review, channel review, summarise day/week | Yes |
| **Data warehouse MCP** (e.g. Snowflake, BigQuery) | Daily metrics, customer prep | Yes |
| **Meeting notetaker MCP** | Meeting review | Yes |
| **Google Workspace MCP** (Calendar + Gmail) | Calendar review, Gmail review, customer meeting detection | Yes |
| **Task manager MCP** (e.g. Todoist) | Task suggestions | Optional |

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

## Design Principles

- **Opt-in, not automatic:** routines only run when you ask for them via slash commands. No surprise interruptions.
- **Skill-per-step:** every routine step is a standalone skill that knows exactly what to do. Skills can be improved independently and reused outside the routine system.
- **State as YAML:** simple, human-readable, easy to debug. No database needed.
- **Idempotent:** if a session crashes mid-routine, the next session picks up where it left off (incomplete steps stay `false`).
- **Progressive:** placeholder skills let you define the full routine now and activate steps as you connect new tools.
- **Context-aware:** calendar and email skills detect what matters (customer meetings, urgent emails) and proactively offer deeper preparation.

## Requirements

- [Cursor IDE](https://cursor.com/) with agent mode
- Slack MCP (for most skills)
- A data warehouse MCP (for metrics)
- A meeting notetaker MCP (for meeting review)
- Google Workspace MCP (for Gmail and Calendar)

## Licence

MIT
