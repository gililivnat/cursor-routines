# Week Summary

Run on **Friday** to summarise the week's activity, metrics, and meetings.

---

## Skill

**Skill:** `skills/summarise-week/SKILL.md`
**State field:** `weekly.summarise_week`

## What It Does

Consolidates the week's work into a single summary covering:
- What shipped and key Slack discussions
- Product metrics trends (from daily metrics reviews)
- Meeting outcomes and action items (from notetaker digests)
- Carry-over items for next week

Builds on the Product Ops agent's `weekly-product-summary` task but adds a personal layer: his meetings, his Slack threads, his priorities.

---

## Completion

When done, set `weekly.summarise_week` to `true` in `state.md`.
