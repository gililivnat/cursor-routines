# Routines Orchestrator

This is the master control file for the user's daily and weekly routines. The Cursor rule reads this file at the start of every conversation and executes the flag logic below.

---

## State File

`routines/state.md` -- YAML frontmatter tracks the current date, group flags, and per-step completion.

---

## Flag Logic

At the start of every conversation, execute these steps in order:

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
5. Write the updated frontmatter back to `state.md`

**If `date` == today:** proceed to Step 3 without resetting.

### Step 3 -- Determine what to run

Check each group flag:

- **`daily_start.flag == 0`:** morning routines have not completed. Read `routines/daily/start-of-day.md` and run any skills whose state field is still `false`.
- **`daily_end.flag == 0`:** end-of-day routines have not completed. Read `routines/daily/end-of-day.md` and run any skills whose state field is still `false`. Note: only run EOD routines if the user explicitly asks to summarise the day, or if it's clearly the end of the working day.
- **`weekly.flag == 0`:** weekly routines have not completed. Check the day:
  - **Monday:** run `weekly/week-planning.md`
  - **Friday:** run `weekly/week-summary.md`
  - **Last month of quarter:** flag `weekly/quarterly-planning.md` as available
  - Run any weekly skills whose state field is still `false` and whose cadence matches today.

**If all flags == 1:** all routines are done for today. Proceed to Step 4 without running routines.

### Step 4 -- Check scheduled tasks

After handling routines, read `your scheduled tasks file` and check for:
- **Recurring weekly (Monday):** flag if today is Monday
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
| -- | Review Upcoming Meetings | `skills/review-upcoming-meetings/SKILL.md` | `review_upcoming_meetings` (placeholder) |
| -- | Review Gmail | `skills/review-gmail/SKILL.md` | `review_gmail` (placeholder) |

### Daily End Skills

| Order | Skill | Path | State Field |
|-------|-------|------|-------------|
| 1 | Summarise Product Day | `skills/summarise-campaigns-day/SKILL.md` | `summarise_campaigns_day` |
| 2 | Review Notetaker Meetings | `skills/review-notetaker-meetings/SKILL.md` | `review_notetaker_meetings` |

### Weekly Skills

| Skill | Path | State Field | Cadence |
|-------|------|-------------|---------|
| Summarise Week | `skills/summarise-week/SKILL.md` | `summarise_week` | Friday |
| Plan Week | `skills/plan-week/SKILL.md` | `plan_week` | Monday |
| Plan Quarter | `skills/plan-quarter/SKILL.md` | `plan_quarter` | Last month of Q |

---

## Presenting Routines to the user

When routines are due, present them as a brief summary:

```
Routines due this session:

**Start of Day** (4 steps remaining)
1. Review Slack DMs
2. Review Product channels
3. Summarise Slack + suggest tasks
4. Review daily metrics

Shall I run through them now?
```

Wait for the user's confirmation before executing. If he says yes, run each skill in order, updating `state.md` after each one completes.

---

## Important Rules

- **Never skip the state check.** Always read `state.md` first.
- **Never run EOD routines in the morning** unless the user explicitly asks.
- **Always update state.md** after each skill completes, not just at the end.
- **Commented-out fields are inactive.** Do not run skills for commented fields. When a new MCP is connected, uncomment the field to activate.
- **Scheduled tasks in `your scheduled tasks file`** are checked separately and are not part of the flag system. They follow their own cadence/due-date logic.
