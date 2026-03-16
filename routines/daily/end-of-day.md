# End of Day

Run these skills at the end of the working day, or when the user asks to summarise the day. Each step reads its SKILL.md, executes, and updates `state.md` when done.

---

## Steps

### 1. Summarise Product Day

**Skill:** `skills/summarise-campaigns-day/SKILL.md`
**State field:** `daily_end.summarise_campaigns_day`

Review product-related Slack messages from today. Produce a summary with highlights, key decisions, and anything needing follow-up.

---

### 2. Review Notetaker Meetings

**Skill:** `skills/review-notetaker-meetings/SKILL.md` (existing skill)
**State field:** `daily_end.review_notetaker_meetings`

Check the AI notetaker for today's meetings. Summarise what was discussed, extract action items, and flag unresolved topics.

---

## Completion

When all steps are done, the orchestrator sets `daily_end.flag` to `1` in `state.md`.
