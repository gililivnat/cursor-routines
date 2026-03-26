---
date: "2026-01-01"
daily_start:
  flag: 0
  review_slack_dms: false
  review_campaigns_channels: false
  summarise_slack_suggest_tasks: false
  review_daily_metrics: false
  weekly_competitor_intel: true        # Monday only — skipped (Thursday)
  review_gmail: false
  review_upcoming_meetings: false
daily_end:
  flag: 0
  summarise_campaigns_day: false
  review_notetaker_meetings: false
  review_gmail_eod: false
  review_todoist_tasks: false
weekly:
  week_of: "2026-03-23"
  flag: 0
  summarise_week: false
  plan_week: false
  plan_quarter: false
monday_tasks:
  week_of: "2026-03-23"
  flag: 0
  review_ask_campaigns: false
  review_campaigns_churns: false
  sync_external_skills: false
biweekly:
  last_run: ""
  flag: 0
  prepare_biweekly_update: false
---

# Routine State

This file tracks whether daily and weekly routines have been completed. The orchestrator reads and updates it automatically.

**Do not edit manually** unless resetting state (e.g. setting all flags to 0 to force a re-run).

## How It Works

- `date`: the date this file was last checked. If it's older than today, all daily flags reset.
- `flag`: group-level completion flag. `1` means all steps in that group are done. `0` means at least one step remains.
- Individual fields (e.g. `review_slack_dms`): per-step completion. `true` when the skill has finished.
- `week_of`: the Monday of the current week. Resets weekly flags when a new week begins.
- `monday_tasks`: tracks Monday-only recurring scheduled tasks. Resets weekly like `weekly`.
- Commented-out fields are placeholders for future MCP integrations.
