---
name: "learnings-extractor"
description: |
  Mandatory end-of-agent skill. Every pipeline agent (backend, frontend, devops, helper, researcher) MUST invoke this before reporting completion to the lead. Writes today's raw learnings file at `local/agents/<agent>/learnings/YYYY-MM-DD.md`.

  Use when:
  - You are an agent finishing a non-trivial task.
  - The lead is wrapping a session and wants to capture cross-cutting observations before `/produce-result`.
model: sonnet
---

# learnings-extractor — Raw Per-Agent Daily Learnings

## Purpose

Capture **architecture and code concepts** — things a future agent would search for in `knowledge/`. Narrative ("I tried X, then Y") goes nowhere; only the distilled concept survives.

This file is the **raw archive**. The `/produce-result` skill (lead-only, session end) reads these files, compresses across agents into `knowledge/learnings.md`, and promotes standing rules (to the app repo `CLAUDE.md` gotchas or `knowledge/global.md`).

## Where to write

```
local/agents/<agent-name>/learnings/YYYY-MM-DD.md
```

- `<agent-name>` — your registered name (`backend`, `frontend`, `devops`, `helper`, `researcher`, `lead`).
- One file per agent per day; **append** if it exists.
- The path is gitignored — agents never read these files to start new tasks. Only readers: this skill (dedupe) and `/produce-result`.

## What to write — concept-only contract

One bullet per entry:

```markdown
- **<concept>** — <one-line essence>. Context: <module / file>. Why it matters: <future-you needs this when …>.
```

Allowed: architecture decisions + rationale; newly observed patterns; reusable solutions not obvious from the code; surprises (platform/library quirks, edge cases); non-obvious couplings between config files.
Disallowed: narrative, status updates, anything derivable from reading current code, opinions without a generalisable lesson.

## Workflow

1. Skim your last ~7 day-files under `local/agents/<agent>/learnings/`. A would-be duplicate → update the older entry instead (bump its date marker).
2. Distil ≤ 5 entries per session. More than 5 = you're writing narrative; compress harder.
3. Write/append:
   ```markdown
   # Learnings — <agent-name> — YYYY-MM-DD

   ## Session: <session-slug-or-"adhoc">

   - **<concept>** — <essence>. Context: <where>. Why it matters: <when>.
   ```

## Quality checks

- [ ] One bullet per entry, concrete concept, Context + Why-it-matters present.
- [ ] No narrative. ≤ 5 entries, ≤ 60 lines appended.
- [ ] Date in path = today's local date.
