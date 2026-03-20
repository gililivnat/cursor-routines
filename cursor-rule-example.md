# Example Cursor Rule

Save this as `.cursor/rules/routines.mdc` in your project (adjust the frontmatter format to match your Cursor version).

```
---
description: Routines are opt-in via slash commands. Detect slash commands and run the corresponding routine.
alwaysApply: true
---

# Routines — Opt-In via Slash Commands

Routines are **not automatic**. They only run when the user types a slash command.

## Slash Commands

| Command | What it does |
|---------|--------------|
| `/goodmorning` | Runs the **start-of-day** routine |
| `/goodnight` | Runs the **end-of-day** routine |
| `/planweek` | Runs the **week planning** routine |
| `/weekreview` | Runs the **week summary** routine |
| `/planquarter` | Runs the **quarterly planning** routine |
| `/checktasks` | Checks for due or overdue scheduled tasks |
| `/routines` | Shows a summary of all available slash commands |

## When a Slash Command Is Detected

1. Read `[path-to]/routines/ORCHESTRATOR.md` for the full execution rules
2. Read `[path-to]/routines/state.md` to check the current state
3. Execute the date/reset logic from the orchestrator (reset flags if date < today)
4. Run the routine corresponding to the command
5. Update `state.md` after each skill completes

Each routine step is a skill. Read the skill file and follow its workflow.

## When `/routines` Is Called

Display all available slash commands and their descriptions.

## If No Slash Command Is Detected

Do **not** check routines, do **not** read state.md, do **not** mention routines. Proceed normally.
```

Replace `[path-to]` with the actual path to your routines folder in your workspace.
