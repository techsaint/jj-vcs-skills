# Run file template (claim ticket)

Copy to `plans/<plan>/runs/<PLAN_RUN_ID>.md` **at run start** (status
`in-progress`), then update `status` and the log as the run proceeds. The
filename **is** the `PLAN_RUN_ID` (`<YYYYMMDDTHHMMZ>-<6char>`), which also matches
the workspace directory name. See `docs/decisions/2026-06-19-plan-run-files-and-claim-tickets.md`
for the rationale and `plans/README.md` ("Run files") for the convention.

Liveness is **derived** — record `started`; check aliveness with `jj op log` in
the named workspace(s), not a hand-kept heartbeat. Mirror the "What this run is
doing" line into the workspace's `jj desc` so `jj workspace list` / `jj log` is
self-explaining (a pointer, not a second copy of state).

```markdown
---
plan: <plan-name>
plan_run_id: <YYYYMMDDTHHMMZ>-<6char>   # == this filename
status: in-progress                     # in-progress | passed | abandoned | promoted | rejected
started: <YYYY-MM-DDThh:mmZ>
session: <session id / who>
slices: [<scope of this run, e.g. S0, S1>]
isolation_mode: jj-workspace            # jj-workspace | git-worktree
workspaces:
  - repo: <root | nested-repo-name>     # e.g. flutter_ai_chat
    path: <.workspaces/social_media_implemenation/plans/<plan>/<PLAN_RUN_ID>/...>
    liveness: jj op log -R <path>       # derived; not a hand-kept heartbeat
promotion:
  target_branch: <branch | none yet>
  approval: not requested               # not requested | requested | approved by <who> @ <when> | promoted
---

# Run <PLAN_RUN_ID> — <plan-name>

## What this run is doing
<one paragraph. Mirror the first line into the workspace `jj desc -m "..."`.>

## Shared branch visibility
- repo:
- branch:
- branch changed: yes/no
- old HEAD:
- new HEAD:
- visible to other sessions: yes/no
- export/promotion approval: not requested / requested / approved by <who> at <when>

## Log
- <YYYY-MM-DDThh:mmZ> claimed; workspace created at <path>
- ...
```
