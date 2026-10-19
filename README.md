# basic-proj-structure — reusable Claude Code workspace template

A project-independent copy of the multi-agent workspace structure (lead-orchestrated
pipeline, per-task sessions, learnings capture). Copy it into a new workspace root and
fill in the placeholders.

## What's inside

```
.
├── AGENTS.md                     workspace root doc — agent registry, layout, rules (TEMPLATE)
├── .gitignore                    workspace gitignore (repositories/ and local/ excluded)
├── .claude/
│   ├── agents/                   agent roster: lead, researcher, backend, frontend, devops, helper
│   ├── skills -> ../knowledge/skills   symlink — exposes workspace skills to Claude Code
│   └── settings.local.json       minimal permission allowlist
├── knowledge/
│   ├── global.md                 cross-agent protocol, loaded by every agent (TEMPLATE)
│   ├── learnings.md              curated notes — starts empty, written only by /produce-result
│   ├── services/                 architecture snapshots per service (empty)
│   └── skills/
│       ├── learnings-extractor/  mandatory end-of-task learnings capture (generic)
│       └── produce-result/       lead-only session wrap (generic)
├── sessions/                     one folder per task: task.md, coding-report.md, result.md
├── local/                        gitignored scratch + raw agent learnings day-files
└── repositories/                 gitignored sub-repos (each its own .git)
```

## How to adopt in a new project

1. Copy everything to the new workspace root: `cp -R basic-proj-struture/ <new-root>/`
   (use `cp -R`, not Finder, so the `.claude/skills` symlink survives; verify with
   `ls -la <new-root>/.claude`).
2. Create the `CLAUDE.md` symlink at the root: `ln -s AGENTS.md CLAUDE.md`
   (symlink so Claude Code and other agent tools read the same file).
3. Clone/create sub-repos under `repositories/` and give each its own `CLAUDE.md`
   with stack details + gotchas — agents are instructed to read it before touching code.
4. Search for `<...>` placeholders and `TODO` markers in:
   - `AGENTS.md` — product one-liner, sub-repo table, deploy model, skill list
   - `knowledge/global.md` — build discipline, project credential locations
   - `.claude/agents/*.md` — each coder's owned surface, stack gotchas, validation commands
5. Delete the agents you don't need (e.g. no infra → delete `devops.md`) and remove them
   from the registry tables in `AGENTS.md` and `.claude/agents/lead.md`.
6. Add project-specific skills under `knowledge/skills/<name>/SKILL.md` as needed
   (example from the origin project: `write-database-migration` — enforced idempotent SQL
   migrations). They're auto-exposed via the `.claude/skills` symlink.
7. `git init` at the root, stage everything, commit. `repositories/` and `local/` stay
   gitignored.

## The model in one paragraph

The workspace root never auto-orchestrates. For any non-trivial task the user starts the
**lead** agent, which formalises the request, creates `sessions/<slug>/`, investigates the
code, briefs specialist coder agents with self-contained briefs, reviews their diffs with
`/code-review`, and wraps with `/produce-result`. Every agent writes an artifact under
`sessions/<slug>/` and extracts ≤5 concept-only learnings via `/learnings-extractor` to
`local/agents/<name>/learnings/`. Agents stage (`git add`) but NEVER commit or push — the
user does both manually. Trivial one-shot edits skip the lead: dispatch a single agent
directly.

## Invariants worth keeping (what makes this work)

- **Artifact per step** — missing `sessions/<slug>/` file = the step didn't happen.
- **Lead edits nothing** — every file change is delegated; keeps briefs honest.
- **Coders never dispatch each other** — cross-layer scope is reported back to the lead.
- **Stage, never commit** — the user is the only one who commits/pushes.
- **Learnings pipeline** — raw day-files (gitignored) → compressed `knowledge/learnings.md`
  → promoted standing rules (repo CLAUDE.md gotchas / global.md protocol).
- **Briefs name concrete files** — the lead reads code before dispatching; a vague brief
  produces vague work.
