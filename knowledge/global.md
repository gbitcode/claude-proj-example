# Global Rules — All Agents

These rules apply to the lead and every pipeline agent (`backend`, `frontend`, `devops`, `helper`, `researcher`). Loaded once at session start; never overridden by agent-specific prompts.

Workspace layout and the agent registry live in the root `AGENTS.md`. This file is pure cross-agent protocol.

---

## 1. Request formalisation

Before executing any external request:

**FORMALISE** — `TASK: <verb> <what> in <where> so that <why>. SCOPE: <bounds>. OUTPUT: <deliverable>.`
**CLARIFY + PLAN** — identify needed files/decisions; ambiguity → ask BEFORE execution. Never re-ask something already answered.
**EXECUTE** — only after the first two. New blockers → back to clarify.

Skip only for literal acknowledgements (`ok`, `yes`, `thanks`).

## 2. Simplest working solution first

1. What is the most direct approach? An if-else beats an event system with one listener.
2. Do the pattern's assumptions match the situation? If a cookbook needs workarounds to fit, it's wrong.
3. Symptom or root cause? The same workaround appearing twice means the foundation is wrong.

Red flags: justifying why complexity is "necessary", shims between your solution and the existing system, solving problems that haven't happened yet.

## 3. Load project context before coding

Before editing `repositories/<app-repo>`, read its `CLAUDE.md` — it carries the stack constraints and hard-won gotchas. Re-introducing a listed gotcha is the #1 failure mode in any codebase that keeps one.

## 4. Git discipline

- **Stage, never commit, never push.** `git add` in the right repo is autonomous and expected for new/changed files. `git commit` and `git push` are user-only<!-- keep this clause if push auto-deploys: — a push to `main` auto-deploys to production -->. If asked to draft a commit message, draft it in the report; don't run the commit.
- No `cd <path> && <cmd>` compounds — use `git -C <path> ...`, `npm --prefix <path> ...`, full paths for everything else.
- Never work on a branch the user didn't ask for.
- Commit message style (when drafting for the user): short imperative subject, no trailers, no AI attribution.

## 5. Build discipline (app repo)

<!-- TODO: state what "complete change" means for this project. Origin-project example:
Deploy artifacts are committed. Any source change is INCOMPLETE until the build has run and the regenerated artifacts are staged alongside the source. Never hand-edit generated files. -->
A change is INCOMPLETE until <build/test command> has run and its outputs (if committed) are staged alongside the source.

## 6. Safety

- Never overwrite user changes outside the requested task; treat unfamiliar files as in-progress work.
- Ask before destructive actions, dependency upgrades, or anything touching production data (<list live services: DB, object storage, platform config>).
- <credential file locations, e.g. `repositories/<app-repo>/.env`> hold live credentials — read when needed, never print values into reports or commit them anywhere.
- A repo that holds a `.env` (or any credential file) must have a matching `.gitignore` staged BEFORE its first commit — verify with `git -C <repo> check-ignore -v .env` and `git -C <repo> status --short` (no `??` on the secrets file). Applies even to local-only repos.
- A read-only audit agent pointed at production must be told explicitly which controls mutate state — navigation, screenshots and display-only modals are allowed; anything that creates, deletes, toggles or impersonates is not.

## 7. Progress reporting

Every agent produces its artifact under `sessions/<slug>/`:
- lead: `task.md` (formalised request + plan), `result.md` (session wrap)
- backend / frontend / devops: `coding-report.md` (files changed, build result, verification)
- researcher: `research-<topic>.md`

Reports are plain and concise: what changed, per file, one line each; build/validation outcomes; cross-layer scope discovered (report it — never dispatch other agents yourself).

## 8. Parallel dispatch and deploy order

- Freeze the API contract in `sessions/<slug>/task.md` BEFORE dispatching agents in parallel. Coders build against that file, not against each other; a cross-layer field-name or shape question goes back to the lead, never agent-to-agent.
- Before building anything around a file another agent is editing, re-read it — briefs say "X is editing this now" precisely because its shape changes under you.
- A release that ships both a DB migration and rebuilt deploy artifacts runs the migration FIRST. Prove compatibility with the deployed code by running the committed artifacts from `git HEAD` against the migrated schema, rather than reasoning about column nullability.
- Local tooling that writes or truncates must refuse a non-local `DATABASE_URL` (or equivalent) before it opens a connection. A dev harness that falls back to the repo's live-credential `.env` is a production footgun.
