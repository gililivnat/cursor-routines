# Bi-weekly Update Prep

Run this routine 1-2 days before the Thursday bi-weekly director update meeting. Triggered manually via `/biweekly`.

---

## Steps

### 1. Prepare Bi-weekly Update

**Skill:** `skills/prepare-biweekly-update/SKILL.md`
**State field:** `biweekly.prepare_biweekly_update`
**Owner:** Product Ops agent

Reviews the latest deck from the Google Drive archive, pulls current metrics, scans Slack for features/PRs/research from the last 2 weeks, and drafts a prep message to `#monday-campaigns` asking each team member to update their slides.

---

## Completion

When the step is done, the orchestrator sets `biweekly.flag` to `1` in `state.md`.
