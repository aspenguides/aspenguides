# Repository Guidelines

This is a simple static website that helps guests of aspensuites resort to find useful accomodation restourants and entertainment. For now the site will be static - withotu backend. Servin:

## Structure

| Path                  | What it is                                                                                                                                                |
| --------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `apps/web`            | Vite + TypeScript web app                                                                                                                                 |
| `apps/mobile`         | Expo (React Native) mobile app                                                                                                                            |
| `services/api`        | FastAPI backend — domain modules under `app/`, Alembic migrations, worker entrypoints (`weather_watch.py`, `dispatch_reminders.py`, `account_cleanup.py`) |
| `services/analysis`   | Redis Streams worker for async plant image analysis                                                                                                       |
| `services/classifier` | ML classifier service (PlantCLEF ViT)                                                                                                                     |
| `packages/api-client` | Shared TypeScript API client                                                                                                                              |
| `docs/`               | Decisions, plans, runbooks (indexed by `docs/README.md`)                                                                                                  |

Brand assets in `docs/brand/brand-kit-export/` are self-contained `.dc.html`
files — inline styles only, do not refactor.

## Architecture guardrails (authoritative — do not deviate)

The stack is decided. Do not introduce new infrastructure components,
frameworks, or external services without explicit human approval in the spec:

- **Queues/events**: Redis Streams (conventions in `docs/events.md`). No
  Kafka, RabbitMQ, SQS, Celery, or new brokers.
- **Database**: Postgres via SQLAlchemy 2.0 + Alembic. No other datastores.
- **Cache**: Redis.
- **Object storage**: MinIO (S3 API).
- **Auth**: Clerk. Entitlements (Free/Trial/Pro) are enforced in the backend.
- **Backend**: Python 3.13, FastAPI, Pydantic, uv/ruff/ty (follow the
  python-backend skill). **Frontend**: TypeScript, Vite (web), Expo (mobile).
- **LLM calls** go through the LiteLLM gateway; never call providers directly.
- **Email**: Resend, behind the `EmailSender` protocol in
  `app/notifications/`; only `app/notifications/email.py` may talk to the
  provider. No SMTP libraries, no other providers.
- **Forum**: NodeBB is separate, integrated only via session-handoff SSO.
- **Deploy**: images built in CI → Harbor → tag bump in `vsrv-gitops` →
  Argo CD. This repo contains no k8s manifests; do not add any.

## Commands

mise is the runner: `mise run setup` / `mise run dev` / `mise run test`.
Narrower loops: `pnpm --dir apps/web test`; per Python service (`cd` into it):
`uv run pytest`, `uv run ruff check .`, `uv run ty check`. Use the
`yardling:verify` skill to verify a change end-to-end in the running stack.

Agent-side frontend checks: use `pnpm --dir apps/web test:unit` and `build`
(the typecheck) rather than `test`, which chains `lint` in first; and avoid
`pnpm --dir apps/mobile lint` (`expo lint`), which bootstraps ESLint and
rewrites `pnpm-lock.yaml`, `package.json`, and an eslint config as a side
effect. Revert those files if it runs.

`apps/web` component tests run in vitest's default `node` environment (no
jsdom, no testing-library): they are `renderToStaticMarkup` checks of a
component's initial render. Anything behind a click or state change is
tested through a helper in `src/utils/`, not by simulating events.

## Style & testing

- Match the closest existing module — structure, naming, error handling, and
  test style. Keep changes focused; small patches over broad refactors.
- Every behavior change gets tests; bug fixes get a regression test when
  practical. If automation can't cover it, document manual verification in
  the PR.
- No plaintext secrets, API keys, or model tokens in the repo (SOPS for
  anything encrypted).
- `apps/web` enforces `react-refresh/only-export-components` as a lint
  error: a `.tsx` component file may export only components (and types).
  Pure helpers shared with tests go in `src/utils/`, not the component file.

## Commits & pull requests

Short imperative commits (`Add garden feed pagination`). Agent branches are
named `agent/<slug>`. PRs include a summary, testing performed, and
screenshots for user-visible changes. Stacked PRs use the `gh stack`
extension: create each PR with `gh pr create --base <parent-branch>` (note
the dependency in the body), then wire them together with
`gh stack link <bottom-PR> ... <top-PR>` — or manage the chain end-to-end
with `gh stack init`/`add`/`submit`. `link` creates the stack on GitHub only;
run `gh stack checkout <stack#>` afterwards for local tracking
(`view`/`sync`/`rebase`). Merge stacks bottom-up. Before editing,
run `git status --short` and avoid overwriting unrelated user changes.
Commits are signed through 1Password: `git commit`/`git push` must run
outside the agent sandbox with the 1Password app unlocked, or signing fails
with "failed to fill whole buffer". Never disable signing to work around it.

## Collective memory

Two stores, split by kind:

- **Conventions, architecture decisions, and guardrails** live in `AGENTS.md`
  and `docs/` — they must load automatically for every agent and be
  PR-reviewable. When you establish a convention or find a stale doc, fix it
  in the same PR.
- **Operational gotchas and task-scoped facts** go to `bd remember`.

Work tracking is split the same way: GitHub Issues are the human intake
surface (QA reports, feature requests — filed and tracked by people); beads
are the agent execution ledger.

- Whenever the user mentions a GitHub issue in any form (`#N`, "issue N", an
  issue URL), fetch it with `gh issue view N --comments` before responding —
  never answer from the issue number alone or ask the user to paste it.
- When that issue leads to planned work, create a fully-specced bead for it
  noting `gh-N` (check first that one doesn't already exist), and include
  `Closes #N` in the eventual PR so the issue closes on merge for whoever
  filed it. The issue is a symptom or wish; the bead is the diagnosed spec.
- Agent-internal work (refactors, chores) lives only in beads.

## Beads Issue Tracker

Claude implementation work uses `bd prime` as the compact lifecycle guide
(the `.claude/settings.json` SessionStart hook injects `.beads/PRIME.md`).
Codex is primarily a review agent: inspect Beads only when relevant and do not
create, claim, update, or close tasks during a review unless explicitly asked.
Its Beads hooks are intentionally absent; do not run `bd setup codex` or
`bd setup claude` — this section is maintained by hand, not by `bd`.
Use `bd` rather than markdown TODOs for durable work and `bd remember` rather
than `MEMORY.md` for operational facts. Do not commit, push Git, or sync Dolt
without explicit authority.

<!-- BEGIN BEADS INTEGRATION v:1 profile:minimal hash:6cd5cc61 -->
## Beads Issue Tracker

This project uses **bd (beads)** for issue tracking. Run `bd prime` to see full workflow context and commands.

### Quick Reference

```bash
bd ready              # Find available work
bd show <id>          # View issue details
bd update <id> --claim  # Claim work
bd close <id>         # Complete work
```

### Rules

- Use `bd` for ALL task tracking — do NOT use TodoWrite, TaskCreate, or markdown TODO lists
- Run `bd prime` for detailed command reference and session close protocol
- Use `bd remember` for persistent knowledge — do NOT use MEMORY.md files

**Architecture in one line:** issues live in a local Dolt DB; sync uses `refs/dolt/data` on your git remote; `.beads/issues.jsonl` is a passive export. See https://github.com/gastownhall/beads/blob/main/docs/SYNC_CONCEPTS.md for details and anti-patterns.

## Agent Context Profiles

The managed Beads block is task-tracking guidance, not permission to override repository, user, or orchestrator instructions.

- **Conservative (default)**: Use `bd` for task tracking. Do not run git commits, git pushes, or Dolt remote sync unless explicitly asked. At handoff, report changed files, validation, and suggested next commands.
- **Minimal**: Keep tool instruction files as pointers to `bd prime`; use the same conservative git policy unless active instructions say otherwise.
- **Team-maintainer**: Only when the repository explicitly opts in, agents may close beads, run quality gates, commit, and push as part of session close. A current "do not commit" or "do not push" instruction still wins.

## Session Completion

This protocol applies when ending a Beads implementation workflow. It is subordinate to explicit user, repository, and orchestrator instructions.

1. **File issues for remaining work** - Create beads for anything that needs follow-up
2. **Run quality gates** (if code changed) - Tests, linters, builds
3. **Update issue status** - Close finished work, update in-progress items
4. **Handle git/sync by active profile**:
   ```bash
   # Conservative/minimal/default: report status and proposed commands; wait for approval.
   git status

   # Team-maintainer opt-in only, unless current instructions forbid it:
   git pull --rebase
   git push
   git status
   ```
5. **Hand off** - Summarize changes, validation, issue status, and any blocked sync/commit/push step

**Critical rules:**
- Explicit user or orchestrator instructions override this Beads block.
- Do not commit or push without clear authority from the active profile or the current user request.
- If a required sync or push is blocked, stop and report the exact command and error.
<!-- END BEADS INTEGRATION -->

<!-- BEGIN BEADS CODEX SETUP: generated by bd setup codex -->
## Beads Issue Tracker

Use Beads (`bd`) for durable task tracking in repositories that include it. Use the `beads` skill at `.agents/skills/beads/SKILL.md` (project install) or `~/.agents/skills/beads/SKILL.md` (global install) for Beads workflow guidance, then use the `bd` CLI for issue operations.

### Quick Reference

```bash
bd ready                # Find available work
bd show <id>            # View issue details
bd update <id> --claim  # Claim work
bd close <id>           # Complete work
bd prime                # Refresh Beads context
```

### Rules

- Use `bd` for all task tracking; do not create markdown TODO lists.
- Run `bd prime` when Beads context is missing or stale. Codex 0.129.0+ can load Beads context automatically through native hooks; use `/hooks` to inspect or toggle them.
- Keep persistent project memory in Beads via `bd remember`; do not create ad hoc memory files.

**Architecture in one line:** issues live in a local Dolt DB; sync uses `refs/dolt/data` on your git remote; `.beads/issues.jsonl` is a passive export. See https://github.com/gastownhall/beads/blob/main/docs/SYNC_CONCEPTS.md for details and anti-patterns.
<!-- END BEADS CODEX SETUP -->
