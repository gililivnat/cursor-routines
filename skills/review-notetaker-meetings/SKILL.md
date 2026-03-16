---
name: review-notetaker-meetings
description: Reviews AI notetaker meetings for the day. Summarises discussions, extracts action items, flags unresolved topics. Use as part of the daily end-of-day routine.
---

# Review Notetaker Meetings

## When to Use

This skill is called by the end-of-day routine (step 2). It can also be run independently when asked to review meeting notes.

## Tools

### Meeting Notetaker MCP

This skill works with any AI notetaker that provides an MCP interface (e.g. monday AI notetaker, Fireflies, Otter). Adapt tool names to your setup.

| Tool | Purpose |
|------|---------|
| Meeting retrieval tool | List and retrieve recent meetings |
| Meeting search tool | Search by title, participant, or date |

### Typical Parameters

| Parameter | Purpose |
|-----------|---------|
| `include_summary` | Include AI-generated summary |
| `include_topics` | Include discussion topics |
| `include_action_items` | Include extracted action items |
| `include_transcript` | Include full transcript (use sparingly, large payload) |

## Workflow

### Step 1 -- Retrieve today's meetings

Query for meetings from today. Include summaries and action items.

### Step 2 -- Synthesise output

For each meeting, extract:
- Summary of what was discussed
- Key decisions made
- Action items with owners
- Open questions or unresolved topics

### Step 3 -- Consolidate across meetings

If there were multiple meetings:
- Group action items by owner
- Identify recurring themes or topics
- Flag unresolved items needing follow-up

### Step 4 -- Produce summary

## Output

### Single meeting

```
**[Meeting title]** | [date]
**Participants:** [list]

**Summary:** [AI summary]

**Decisions:**
- [Decision made]

**Action items:**
- [Owner]: [action]

**Open questions:**
- [Unresolved item]
```

### Multiple meetings

```
**Meeting Digest** | [date]
**Meetings reviewed:** [N]

**[Meeting title]** | [participants]
- Summary: [1-2 sentences]
- Action items: [Owner]: [action]

---

**Consolidated action items:**
- [Owner]: [action] (from: [meeting title])

**Themes across meetings:**
- [Pattern or recurring topic]
```

## Guidelines

- Only include transcript content if specifically asked (transcripts are large)
- Flag if any meetings couldn't be retrieved or if data seems incomplete
- Be concise: summaries should be scannable
- Draft outputs for review before sending anywhere

## Change Log

| Date | Change |
|------|--------|
| 13 Mar 2026 | Initial version |
| 16 Mar 2026 | Restructured as generic workflow skill |
