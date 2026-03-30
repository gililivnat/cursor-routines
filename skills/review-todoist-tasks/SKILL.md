---
name: review-todoist-tasks
description: Fetches today's tasks and displays them in the EOD summary with Postpone and Open in your task manager buttons. Use as part of the daily end-of-day routine.
---

# Review task manager Tasks

## When to Use

This skill is called by the end-of-day routine (step 4). It can also be run independently when asked to review today's remaining tasks.

## Inputs

- Filter: today's tasks from task manager

## Tools

### task manager MCP (your task manager MCP)

| Tool | Purpose |
|------|---------|
| `get_tasks_list` | Fetch tasks filtered by "today" |
| `update_tasks` | Postpone tasks to tomorrow when the user confirms |

## Workflow

### Step 1 -- Fetch today's tasks

```
Tool: get_tasks_list (your-task-manager-mcp)
Arguments:
  filter: "today"
```

### Step 2 -- Process tasks

For each task returned, extract:
- `id` -- task ID (for task manager URL and postpone action)
- `content` -- task title
- `priority` -- priority level (1 = urgent, 4 = normal)
- `project_id` -- which project it belongs to
- `labels` -- any labels
- `due` -- due date/time details

### Step 3 -- Render in EOD HTML

Append a "Today's Tasks" section to the EOD summary HTML at `your output folder/eod-summary-YYYY-MM-DD.html`.

If the EOD HTML doesn't exist yet (this skill runs before the others or independently), create the HTML using the routine template at `skills/system/routine-html-template.md`.

### Step 4 -- Handle postponements

After the user reviews the HTML and returns to Cursor, if he asks to postpone specific tasks:

```
Tool: update_tasks (your-task-manager-mcp)
Arguments:
  items: [{ "task_id": "[id]", "content": "[unchanged title]", "due_string": "tomorrow" }]
```

## Output

### HTML section for this skill

Add this instruction bar before the tasks section:

```html
<div class="send-instructions">
  <strong>How postponing works:</strong> Click "Postpone" to mark tasks you want moved to tomorrow. Then go back to Cursor and tell the agent to postpone the marked tasks.
</div>
```

Then render each task:

```html
<div class="section">
  <div class="section-title">Today's Tasks</div>

  <!-- Repeat for each task -->
  <div class="task-row" data-task-id="[task_id]">
    <div class="task-content">
      <span class="task-priority" style="color: [priority-colour];">[P1/P2/P3/P4]</span>
      [task content]
    </div>
    <div class="task-actions">
      <button class="btn-postpone" onclick="this.classList.toggle('postponed'); this.innerHTML = this.classList.contains('postponed') ? '<span class=\'icon\'>✓</span> Postponed' : '<span class=\'icon\'>→</span> Postpone'">
        <span class="icon">→</span> Postpone
      </button>
      <a href="https://app.todoist.com/app/task/[task_id]" target="_blank" class="btn-open-todoist">
        <span class="icon">↗</span> Open in your task manager
      </a>
    </div>
  </div>

</div>
```

Priority colour mapping:
- Priority 1 (urgent): `color: #f87171` — P1
- Priority 2: `color: #fbbf24` — P2
- Priority 3: `color: #00d4ff` — P3
- Priority 4 (normal): `color: #666` — P4

If no tasks are due today: `<p class="quiet">No tasks due today. Clean slate.</p>`

## Guidelines

- Show all tasks due today, regardless of project
- Sort by priority (P1 first, P4 last), then alphabetically
- If a task has a specific time, show it next to the content
- Keep the section concise -- task names should speak for themselves
the inline summary

## Change Log

| Date | Change |
|------|--------|
| 26 Mar 2026 | Initial version |
