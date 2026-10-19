# Workspace — <project-name>

This is the multi-repo workspace root for <product one-liner>. It does not auto-orchestrate. Without explicit orchestration you operate as a regular Claude Code session — the user dispatches agents themselves or starts the lead.

If the user asks for anything more complex than a single trivial edit, recommend starting the **`lead`** agent. The lead owns pipeline planning, team dispatch, monitoring, and session wrap. Its prompt is at `.claude/agents/lead.md`.

# Sub-repositories

Each is its own git repo. Treat as separate deployables.

| Repo | Kind |
|---|---|
| `<app-repo>` | The product — <stack one-liner: runtime, language, build, DB, hosting>. Has its own `CLAUDE.md` with stack details + gotchas — every agent touching it MUST read that file first. |
| `<infra-repo>` | <infra-of-record: deploy specs, runbooks, env-var inventory>. <remote or local-only>. |

# Workspace layout

```
<workspace-root>
├── AGENTS.md, CLAUDE.md (symlink → AGENTS.md)
├── knowledge/
│   ├── global.md               cross-agent rules (loaded by every agent)
│   ├── learnings.md            curated short notes (written only by /produce-result)
│   ├── services/               architecture snapshots per service
│   └── skills/<name>/SKILL.md  workspace skills (exposed via .claude/skills symlink)
├── sessions/<slug>/            one folder per task (created by lead): task.md, coding-report.md, result.md
├── local/                      gitignored: scratch, archives, agents/<name>/learnings/ raw day-files
├── repositories/               sub-repos (gitignored; each has its own .git)
└── .claude/                    agents/ roster, skills → ../knowledge/skills
```

# Skills

Stored under `knowledge/skills/<name>/SKILL.md`, invoked via `/<name>`:

- `learnings-extractor` — MANDATORY for every agent at task end. Raw concept-only day-file to `local/agents/<agent>/learnings/YYYY-MM-DD.md` (≤5 bullets, Context + Why-it-matters, no narrative).
- `produce-result` — lead-only at session wrap: `sessions/<slug>/result.md`, compresses day-files into `knowledge/learnings.md`, promotes standing rules (app gotchas → app repo CLAUDE.md; workspace protocol → knowledge/global.md).
- <!-- TODO: add project-specific skills here (e.g. a migration-writing skill, a release checklist) -->

# Agents

Stored under `.claude/agents/`. The **lead** is started manually by the user; the others are dispatched BY the lead (or directly for one-shot trivial edits).

| Agent | Role | Model |
|---|---|---|
| `lead` | Orchestrator — formalise, plan, brief, dispatch, monitor, wrap. Read+exec, NO file edits. Started manually. | opus |
| `researcher` | Heavy research / task-definition prep. Read-only on code. | opus |
| `backend` | All server-side code in `repositories/<app-repo>`: <owned dirs>. | opus |
| `frontend` | Public web surface: <owned dirs>, anything rendered in a browser. | sonnet |
| `devops` | Deploy/platform config, env wiring, ops tooling, the `<infra-repo>` repo. | sonnet |
| `helper` | Workspace meta: `.claude/`, `knowledge/`, root docs — anything outside `repositories/`. | sonnet |

There is no reviewer agent — the lead runs the `/code-review` skill after coding rounds.

# Deploy model

<!-- TODO: describe how this product ships. Keep the push=deploy warning if it applies. Origin-project example:
- Pre-built artifacts are COMMITTED to the app repo; after any source change the build must run and refreshed artifacts staged together with the source.
- `git push` to `main` auto-deploys. Therefore push = deploy. Push only on explicit user command — and the user normally commits manually (agents stage only).
- Platform spec (domains, ingress, env vars) canonical copy lives in the infra repo, applied via dashboard/CLI — not deployed by git push. -->

# Quick rules (apply always)

- Read `knowledge/global.md` before doing anything; read `repositories/<app-repo>/CLAUDE.md` before touching the app repo.
- No `cd <path> && <cmd>` compounds — use `git -C <path> ...`, `npm --prefix <path> ...`, full paths otherwise.
- Stage new/changed files (`git add`) in the right repo, but NEVER commit and NEVER push — the user does both manually. Zero exceptions.
- Every pipeline step produces an artifact under `sessions/<slug>/`. Missing artifact = step didn't happen.
- Simplest working solution first.
- No AI/Claude attribution in any commit message the user asks you to draft.

# How to use this workspace

1. User starts the **`lead`** agent for any non-trivial task.
2. Lead formalises the request, proposes a roster, spawns teammates with self-contained briefs.
3. Coders edit + build + validate, write `sessions/<slug>/coding-report.md`. They never commit.
4. Lead runs `/code-review` on the diff, loops fixes back to coders if needed.
5. Lead writes `sessions/<slug>/result.md`, stages everything, and stops — the user commits/pushes.

For a one-shot trivial edit, dispatch a single agent directly — no need for the lead.
