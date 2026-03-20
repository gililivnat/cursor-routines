# Routines Orchestrator

This is the master control file for the user's daily and weekly routines. Routines are **opt-in** and only run when the user triggers them via slash commands.

---

## Slash Commands

| Command | Routine | Reference |
|---------|---------|-----------|
| `/goodmorning` | Start of day | `daily/start-of-day.md` |
| `/goodnight` | End of day | `daily/end-of-day.md` |
| `/planweek` | Week planning | `weekly/week-planning.md` |
| `/weekreview` | Week summary | `weekly/week-summary.md` |
| `/planquarter` | Quarterly planning | `weekly/quarterly-planning.md` |
| `/mondaytasks` | Monday recurring tasks | Step 4 below |
| `/checktasks` | Check scheduled tasks | Step 4b below |
| `/voc` | Voice of the Customer | `skills/voice-of-the-customer/SKILL.md` |
| `/routines` | Show all commands | Display command list |

---

## State File

`routines/state.md` -- YAML frontmatter tracks the current date, group flags, and per-step completion.

---

## Execution Logic

When a slash command is triggered, execute these steps in order:

### Step 1 -- Read state

Read the YAML frontmatter of `routines/state.md`. Extract `date`, all group flags, and all per-step fields.

### Step 2 -- Check date and reset if needed

Compare `date` to today's date.

**If `date` < today:**
1. Set `date` to today
2. Reset all `daily_start` fields to `false` and `daily_start.flag` to `0`
3. Reset all `daily_end` fields to `false` and `daily_end.flag` to `0`
4. Calculate the Monday of the current week. If `weekly.week_of` < this Monday:
   - Set `weekly.week_of` to this Monday
   - Reset all `weekly` fields to `false` and `weekly.flag` to `0`
5. If `monday_tasks.week_of` < this Monday:
   - Set `monday_tasks.week_of` to this Monday
   - Reset all `monday_tasks` fields to `false` and `monday_tasks.flag` to `0`
6. Write the updated frontmatter back to `state.md`

**If `date` == today:** proceed to Step 3 without resetting.

### Step 3 -- Run the requested routine

Based on the slash command, run the corresponding routine:

- **`/goodmorning`** → Read `routines/daily/start-of-day.md` and run any skills whose state field is still `false`. **Conditional skills:** `weekly_competitor_intel` only runs on Mondays. On other days, treat it as already complete (set to `true` and skip). Gmail (step 6) appends to the morning briefing HTML after the main content. Calendar (step 7) appends today's meetings to the morning briefing HTML and flags any customer meetings, offering to run `prepare-customer-interview` for those accounts. If `daily_start.flag == 1`, tell the user morning routines are already done for today.
- **`/goodnight`** → Read `routines/daily/end-of-day.md` and run any skills whose state field is still `false`. Gmail EOD (step 3) appends email highlights to the EOD summary HTML. If `daily_end.flag == 1`, tell the user end-of-day routines are already done.
- **`/planweek`** → Run `weekly/week-planning.md`. If `weekly.plan_week == true`, tell the user week planning is already done this week.
- **`/weekreview`** → Run `weekly/week-summary.md`. If `weekly.summarise_week == true`, tell the user week summary is already done this week.
- **`/planquarter`** → Run `weekly/quarterly-planning.md`. If `weekly.plan_quarter == true`, tell the user quarterly planning is already done.
- **`/mondaytasks`** → Jump to Step 4 (Monday scheduled tasks).
- **`/checktasks`** → Jump to Step 4b (check scheduled tasks).
- **`/voc`** → Read and execute `skills/voice-of-the-customer/SKILL.md`.
- **`/routines`** → Display the list of all available slash commands and descriptions.

### Step 4 -- Check and track Monday scheduled tasks

Monday recurring tasks are tracked in `state.md` under `monday_tasks`. This works like other routine groups:

**If today is Monday and `monday_tasks.flag == 0`:**

1. Check `monday_tasks.week_of`. If it's before this Monday, reset all fields to `false` and update `week_of`.
2. Present the incomplete Monday tasks to the user and ask which to run.
3. After each task completes, set its field to `true`. When all are `true`, set `monday_tasks.flag` to `1`.

| State field | Task | Owner | Reference |
|-------------|------|-------|-----------|
| `review_ask_campaigns` | Review Ask-Product channel (7-day) | Product Ops agent | `Agents/product-ops-agent.md` |
| `review_campaigns_churns` | Review Product Churns channel (7-day) | Product Ops agent | `Agents/product-ops-agent.md` |
| `sync_external_skills` | Sync external skills repos | Master agent | `your scheduled tasks file` |

**Follow-up actions for specific tasks:**

| Task | If result contains... | Follow-up |
|------|-----------------------|-----------|
| `review_campaigns_churns` | Any churned accounts | Create a task manager task for today: "Send personalised churn feedback emails to [list of account names]". Use the `write-churn-feedback-email` skill (`~/.cursor/skills/write-churn-feedback-email/SKILL.md`) to draft each email. |

**If today is not Monday:** skip `monday_tasks` entirely.

### Step 4b -- Check other scheduled tasks

After handling Monday tasks, read `your scheduled tasks file` and check for:
- **Recurring weekly (Friday):** flag if today is Friday
- **One-off tasks:** flag if today is on or after the due date

If any tasks are due or overdue, alert the user with a brief summary noting which agent owns each task.

### Step 5 -- Update state after each skill

As each skill completes:
1. Set its field to `true` in `state.md`
2. Check if all active (non-commented) fields in the group are now `true`
3. If yes, set the group `flag` to `1`

---

## Routine Files

These files list which skills to run and in what order. They are lightweight orchestration files, not skill definitions.

| Routine | File | When |
|---------|------|------|
| Start of Day | `daily/start-of-day.md` | Every morning (first session of the day) |
| End of Day | `daily/end-of-day.md` | End of working day / when asked |
| Week Summary | `weekly/week-summary.md` | Friday |
| Week Planning | `weekly/week-planning.md` | Monday |
| Quarterly Planning | `weekly/quarterly-planning.md` | Last month of quarter / on demand |

---

## Skills Registry

All routine steps are backed by skills in `skills/`. The agent reads the SKILL.md file before executing each step.

### Daily Start Skills

| Order | Skill | Path | State Field |
|-------|-------|------|-------------|
| 1 | Review Slack DMs | `skills/review-slack-dms/SKILL.md` | `review_slack_dms` |
| 2 | Review Product Channels | `skills/review-campaigns-channels/SKILL.md` | `review_campaigns_channels` |
| 3 | Summarise Slack + Suggest Tasks | `skills/summarise-slack-and-suggest-tasks/SKILL.md` | `summarise_slack_suggest_tasks` |
| 4 | Review Daily Metrics | `skills/review-daily-metrics/SKILL.md` | `review_daily_metrics` |
| 5 | Weekly Competitor Intel | `skills/weekly-competitor-intel/SKILL.md` | `weekly_competitor_intel` (Monday only) |
| 6 | Review Gmail | `skills/review-gmail/SKILL.md` | `review_gmail` |
| 7 | Review Upcoming Meetings | `skills/review-upcoming-meetings/SKILL.md` | `review_upcoming_meetings` |

### Daily End Skills

| Order | Skill | Path | State Field |
|-------|-------|------|-------------|
| 1 | Summarise Product Day | `skills/summarise-campaigns-day/SKILL.md` | `summarise_campaigns_day` |
| 2 | Review Notetaker Meetings | `skills/review-notetaker-meetings/SKILL.md` | `review_notetaker_meetings` |
| 3 | Review Gmail (EOD) | `skills/review-gmail/SKILL.md` | `review_gmail_eod` |

### Weekly Skills

| Skill | Path | State Field | Cadence |
|-------|------|-------------|---------|
| Summarise Week | `skills/summarise-week/SKILL.md` | `summarise_week` | Friday |
| Plan Week | `skills/plan-week/SKILL.md` | `plan_week` | Monday |
| Plan Quarter | `skills/plan-quarter/SKILL.md` | `plan_quarter` | Last month of Q |

---

## Presenting Routines to the user

When a slash command is triggered and there are steps to run, present them briefly:

```
Running start of day (6 steps):
1. Review Slack DMs
2. Review Product channels
3. Summarise Slack + suggest tasks
4. Review daily metrics
5. Review Gmail
6. Review upcoming meetings
```

Then run each skill in order, updating `state.md` after each one completes.

---

## Showing All Commands (`/routines`)

When the user types `/routines`, display:

```
Available routines:

/goodmorning — Start of day (Slack DMs, Product channels, summary + tasks, metrics, competitor intel on Mondays, Gmail, calendar + customer prep)
/goodnight — End of day (summarise campaigns day, review notetaker meetings, Gmail highlights)
/planweek — Week planning (priorities, tasks, upcoming meetings)
/weekreview — Week summary (consolidate week's activity, metrics, meetings)
/planquarter — Quarterly planning (review quarter, draft next quarter priorities)
/mondaytasks — Monday recurring tasks (ask-product, churns, sync skills, check Google MCP)
/checktasks — Check for due or overdue scheduled tasks
/voc — Voice of the Customer report (6-source customer insights)
/routines — Show this list
```

---

## Important Rules

- **Routines are opt-in only.** Never run routines automatically. Only run when a slash command is triggered.
- **Never skip the state check.** Always read `state.md` first when a routine is triggered.
- **Always update state.md** after each skill completes, not just at the end.
- **Commented-out fields are inactive.** Do not run skills for commented fields. When a new MCP is connected, uncomment the field to activate.
- **Scheduled tasks in `your scheduled tasks file`** are checked separately and are not part of the flag system. They follow their own cadence/due-date logic.
- **If no slash command is detected**, do not check routines, do not read state.md, and do not mention routines.
