---
name: onboarding
description: Interactive setup wizard for cursor-routines. Asks questions to configure all skills, generates config.yaml, sets up state.md, installs the Cursor rule, and optionally copies skills to ~/.cursor/skills/. Use when the user says "onboarding", "set up cursor-routines", "configure routines", or "get started".
---

# Onboarding

Interactive setup wizard that configures cursor-routines for your workspace. Walks through 6 phases of questions, writes a `config.yaml`, and applies your answers across all skill files.

## When to Use

- First time setting up cursor-routines
- Re-running after cloning a fresh copy of the repo
- Reconfiguring after changing MCP servers or workspace layout

## Prerequisites

- This repo cloned somewhere in your workspace
- Cursor IDE with agent mode enabled
- At least a Slack MCP connected (the core skills depend on it)

## Workflow

### Phase 1 -- Workspace

Ask the user:

1. **Where is this repo in your workspace?**
   - Detect automatically if possible (look for `routines/ORCHESTRATOR.md` relative to the current workspace root)
   - Examples: repo root, `Tools/routines`, `cursor-routines`
   - This determines all internal path references

2. **Where should output files be saved?**
   - Default: `Output/` relative to workspace root
   - This is where HTML reports, summaries, and plans get saved
   - Create the directory if it doesn't exist

Store as `routines_path` (relative path to repo root) and `output_path`.

### Phase 2 -- Product identity

Ask the user:

3. **What's your product or project name?**
   - Used in report titles, summaries, and context
   - Examples: "Acme Analytics", "our product", "the platform"

4. **What keyword identifies your product's Slack channels?**
   - The system searches for Slack channels containing this keyword
   - Examples: "product", "marketing", "analytics", "platform"
   - This replaces `CHANNEL_KEYWORD` across skills

5. **Who are your main competitors?** (optional, can skip)
   - Comma-separated list
   - Used by competitor intel and VoC skills
   - Examples: "Mailchimp, HubSpot, ActiveCampaign"

Store as `product_name`, `channel_keyword`, `competitors`.

### Phase 3 -- Slack

Ask the user:

6. **What's your Slack user ID?**
   - Needed for DM review, thread tracking, and channel scanning
   - How to find it: click your profile picture in Slack, select "Profile", click the three dots menu, select "Copy member ID"
   - Format: starts with `U`, e.g. `U01ABC2DEF3`
   - This replaces `YOUR_SLACK_USER_ID` across skills

7. **Do you have specific Slack channels for support/feedback and churn?** (optional)
   - Support/feedback channel: where customer questions come in (e.g. "ask-product", "support-feedback")
   - Churn channel: where churn notifications land (e.g. "product-churns", "churn-alerts")
   - If none, the system will search by keyword instead
   - These replace `ask-product` and `product-churns` references

Store as `slack_user_id`, `support_channel`, `churn_channel`.

### Phase 4 -- Data

Ask the user:

8. **Do you have a data warehouse MCP connected in Cursor?** (yes/no)
   - If yes: what's the MCP server name? (e.g. "user-snowflake", "user-bigquery")
   - If no: the daily metrics and data-dependent skills will be noted as inactive

9. **Do you have documentation for your data schema?** (optional)
   - Path to a file describing your tables, metrics, and queries
   - If not, a placeholder file can be created
   - This replaces `your data guide` references

10. **Do you have a product strategy document?** (optional)
    - Path to a file with product strategy, positioning, or OKRs
    - This replaces `your product strategy documentation` references

11. **What are your key product metrics?** (optional)
    - Comma-separated list of metrics to track daily
    - Examples: "DAU, MAU, revenue, activation rate, campaigns sent"
    - Used by the daily metrics skill

Store as `data_warehouse_mcp`, `data_guide_path`, `strategy_doc_path`, `key_metrics`.

### Phase 5 -- Tools

Ask the user:

12. **Do you have a task manager MCP?** (optional)
    - Which one? (e.g. Todoist, Linear, Asana)
    - MCP server name (e.g. "user-todoist")
    - Default project and section names for new tasks
    - If none, task suggestions will be presented but not auto-created

13. **Do you have a meeting notetaker MCP?** (optional)
    - MCP server name (e.g. "user-meeting-notes")
    - Tool name for retrieving meetings (e.g. "get_meetings")
    - If none, the meeting review skill stays as placeholder

Store as `task_manager_mcp`, `task_project`, `task_section`, `notetaker_mcp`.

### Phase 6 -- Optional integrations

Ask the user:

14. **Do you have an analytics platform MCP?** (e.g. Mixpanel, Amplitude)
    - MCP server name and project ID if applicable
    - Used by voice-of-the-customer for behavioural data

15. **Do you have a Calendar MCP connected?**
    - If yes, the review-upcoming-meetings skill will be activated in state.md

16. **Do you have a Gmail MCP connected?**
    - If yes, the review-gmail skill will be activated in state.md

Store as `analytics_mcp`, `analytics_project_id`, `calendar_mcp`, `gmail_mcp`.

---

## Apply Configuration

After collecting all answers, execute these steps in order.

### Step 1 -- Write config.yaml

Write a `config.yaml` file at the repo root with all collected values. Use `config.example.yaml` as the template. This file serves as a reference for future re-runs and makes the configuration git-diffable.

### Step 2 -- Replace placeholders across all skill files

Apply these replacements across every `.md` file in the `skills/` directory:

| Placeholder | Replace with | Source |
|-------------|-------------|--------|
| `YOUR_SLACK_USER_ID` | User's Slack ID | `slack_user_id` |
| `CHANNEL_KEYWORD` | Product channel keyword | `channel_keyword` |
| `ask-product` | Support channel name | `support_channel` (or keep as-is if empty) |
| `product-churns` | Churn channel name | `churn_channel` (or keep as-is if empty) |
| `your data warehouse MCP` | Actual MCP name | `data_warehouse_mcp` (or keep placeholder if empty) |
| `your data guide` | Actual path | `data_guide_path` (or keep placeholder if empty) |
| `your product strategy documentation` | Actual path | `strategy_doc_path` (or keep placeholder if empty) |
| `your output folder` | Output path | `output_path` |
| `your Slack MCP` | `user-slack` or user's MCP name | Usually `user-slack` |
| `your meeting notetaker MCP` | Actual MCP name | `notetaker_mcp` (or keep placeholder if empty) |
| `your meeting retrieval tool` | Actual tool name | from `notetaker_mcp` config |
| `your analytics MCP` | Actual MCP name | `analytics_mcp` (or keep placeholder if empty) |
| `your task manager` | Task manager name | `task_manager_mcp` name (or keep placeholder if empty) |
| `your task creation skill` | Skill name/path | (or keep placeholder if empty) |
| `your tone of voice guide` | Path or keep as-is | (keep placeholder -- user creates their own) |
| `your scheduled tasks file` | Path or keep as-is | (keep placeholder -- user creates their own) |
| `your learnings file` | Path or keep as-is | (keep placeholder) |
| `your decisions file` | Path or keep as-is | (keep placeholder) |

**Important:** Only replace placeholders where the user provided a value. Leave placeholders intact for skipped/empty fields so the user knows what's still unconfigured.

### Step 3 -- Create state.md

Copy `routines/state.example.md` to `routines/state.md` with these adjustments:

1. Set `date` to today's date
2. If Calendar MCP is connected: uncomment `review_upcoming_meetings: false`
3. If Gmail MCP is connected: uncomment `review_gmail: false`
4. If no data warehouse MCP: add a comment noting `review_daily_metrics` will be skipped
5. If no notetaker MCP: add a comment noting `review_notetaker_meetings` will be skipped

### Step 4 -- Generate the Cursor rule

Read `cursor-rule-example.md` from the repo root. Replace `[path-to]` with the actual path to the routines folder relative to the workspace. Save it as `.cursor/rules/routines.mdc` in the user's workspace (ask for confirmation first).

### Step 5 -- Install skills (optional)

Ask the user if they'd like to copy skills to `~/.cursor/skills/` for auto-discovery. If yes:

```bash
for skill in skills/*/; do
  name=$(basename "$skill")
  [ "$name" = "system" ] && continue
  [ "$name" = "onboarding" ] && continue
  mkdir -p ~/.cursor/skills/$name
  cp "$skill/SKILL.md" ~/.cursor/skills/$name/SKILL.md
done
```

### Step 6 -- Create output directories

Create the output directory structure:

```bash
mkdir -p [output_path]/routines
mkdir -p [output_path]/presentations
mkdir -p [output_path]/voice-of-customer
mkdir -p [output_path]/competitor-analysis
mkdir -p [output_path]/customer-interview-prep
mkdir -p [output_path]/weekly-summaries
mkdir -p [output_path]/quarterly-planning
```

### Step 7 -- Print summary

Present a summary showing:

**Configured:**
- List every setting that was filled in with its value

**Still needs setup:**
- List any placeholders that were left empty
- Note which skills are inactive because of missing MCPs

**Next steps:**
- "Start a new Cursor conversation -- the routines system will activate automatically"
- "Say 'start my day' to run through the morning routine"
- "Run 'onboarding' again any time to reconfigure"

---

## Re-running Onboarding

If `config.yaml` already exists when the skill runs, read it first and present the current values to the user. Ask which phase they'd like to reconfigure (or "all" to start fresh). Only overwrite values they explicitly change.

## Guidelines

- Ask questions in batches (one phase at a time), not all at once
- Provide sensible defaults where possible
- Make every question skippable except Slack user ID (which most skills depend on)
- After each phase, briefly confirm what was captured before moving on
- Be conversational, not form-like

## Change Log

| Date | Change |
|------|--------|
| 18 Mar 2026 | Initial version. 6-phase onboarding flow covering workspace, product identity, Slack, data, tools, and optional integrations. |
