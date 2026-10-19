---
name: "produce-result"
description: |
  Lead-only skill. Runs at session wrap to consolidate everything into `sessions/<slug>/result.md`, compress raw agent learnings into `knowledge/learnings.md`, and promote standing rules.

  Use when:
  - The lead finished the pipeline (task, dispatch, review, learnings extracted) and is ready to wrap.
  - The user asks to "wrap this session" / "produce the result file".
---

# produce-result — Session Wrap-up

Single source of truth for what a session produced. The lead delegates the actual file writes to `helper` (lead never edits files) but owns the content.

## Inputs

- `slug` — session slug.
- Reads: `sessions/<slug>/task.md`, `coding-report.md`, `research-*.md`, and `local/agents/*/learnings/<today>.md` for every participating agent.

## Workflow

### Step 1 — write/update `sessions/<slug>/result.md`

RESUME rule: if the file exists, append a new round under `## Notes` and refresh statuses — never reset prior content.

```markdown
# <slug> — <short feature name>

## Outcome
<2-4 lines: what shipped / what was answered>

## Changes
| Repo | Files | Build | Staged |
|---|---|---|---|
| <app-repo> | <paths> | ✅/n-a | ✅ |

## Drafted commit message(s)
- <repo>: `<subject>`

## Notes
- <round-by-round decisions>

## Open items
- <non-blocking leftovers / follow-ups>
```

### Step 2 — read all participating agents' day-files

`local/agents/<agent>/learnings/YYYY-MM-DD.md` (today, workspace local date).

### Step 3 — compress into `knowledge/learnings.md`

Grouped by category (e.g. Runtime / platform, Data model, Config / env, Workflow, History context — add a category only when 3+ entries argue for it).

Per raw entry: match category → compress to one line (imperative voice) → dedupe against existing content (duplicate = no-op; extension = rewrite the existing line to subsume) → drop session-specific narrative (that belongs in `result.md`).

### Step 4 — promote standing rules

A compressed entry that is imperative, applies beyond this session, and isn't recorded yet gets promoted:
- app-specific gotcha → `repositories/<app-repo>/CLAUDE.md` `## Gotchas`
- workspace-wide protocol → `knowledge/global.md`
Keep the line in `learnings.md` too; it graduates naturally.

### Step 5 — report stub to the chat

```
## /produce-result — session <slug>
- result.md: created|updated
- knowledge/learnings.md: <K> new/updated lines in <C> categories
- promotions: <files touched or "none">
- day-files consumed: <list>
```

## Invariants

- Never DELETE lines from `knowledge/learnings.md` (rewrite-to-subsume only; cleanup is a separate manual reconcile).
- Never edit the day-files — they are the audit trail.
- Wrapping ≠ shipping: never commit/push; everything ends staged.
