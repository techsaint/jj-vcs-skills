---
name: porthole-jj
description: Use when piloting or operating Jujutsu (jj) in the Porthole workspace; covers nested-repo boundaries, safe setup, repo-local skills, status checks, and when to use the broader using-jj reference.
---

# Porthole Jujutsu Pilot

## TRIGGER
Load this (with `using-jj` for command translation) **before any plan-execution workspace
op** — whenever the task involves jj workspaces, nested-repo isolation, `jj workspace add`,
run files / claim tickets, promotion to a shared branch, or the `$porthole-jj` reference.
SKIP for read-only inspection that touches no version control.

Use this skill for Porthole-specific `jj` work. For general `jj` command
translation or deeper concepts, also load `$using-jj`.

## Scope

- Use this skill when a plan or user request chooses `jj` for a nested
  implementation repo **or for the root repo** (root is jj-colocated; root
  workspace isolation is supported — see `plans/README.md` "Root workspace
  isolation").
- For plan-driven `jj` work, use the plan-run workspace namespace convention
  below so parallel sessions do not collide, and publish a **run file** (see
  "Run files" below) so the run is discoverable from a cold root checkout.
- **Workspace isolation applies to plan *execution*** — including a root/docs-only
  plan's execution. **Ad-hoc orchestration/doc authoring outside a plan** (a quick
  skill edit, an ADR, answering a question) stays in the root default checkout;
  that is not "executing a plan." Do not isolate it, and do not treat editing docs
  inside a *plan's execution* as exempt from isolation.
- Do not silently *initialize* root `jj`: it is a repo-layout decision needing
  user/plan approval (root is already colocated, so this is largely historical).
- Do not initialize `jj` in a nested repo unless the active plan or user request
  names that repo as part of the work.
- Use colocated mode so existing Git tooling keeps working.
- For plan-driven implementation work, treat workspaces as staging by default.
  Moving a shared nested repo branch is a separate promotion/export phase unless
  the active plan explicitly says the target branch is private to the session or
  the user has approved export during slice exit.

## Preflight

Before edits or `jj` setup, print:

```bash
pwd
git status --short --branch
test -d .jj && jj status || true
```

Also check the relevant nested repo status from the root workspace:

```bash
git status --short --branch
cd flutter_porthole && git status --short --branch
```

For non-Flutter work, replace `flutter_porthole` with the nested repo being
touched.

## Setup Pattern

For a touched nested implementation repo:

```bash
cd /home/techsaint/files/src/social_media/social_media_implemenation/<nested-repo>
jj git init --colocate
jj status
```

After colocated setup, prefer `jj` for mutating version-control operations in
that repo. Use Git for read-only inspection when useful.

## Workspace Pattern

Use workspaces — nested-repo **or root** — never a default checkout, for plan
execution. (A colocated `.jj` repo is not isolation by itself: editing the
`default` workspace, nested or root, is visible to other sessions.) When a plan
chooses `jj`, use a single plan-run namespace as the collision boundary:

```text
PLAN_RUN_ID=<YYYYMMDDTHHMMZ>-<6char>
WORKSPACE_ROOT=/home/techsaint/files/src/social_media/.workspaces/social_media_implemenation/plans/<plan-name>/$PLAN_RUN_ID
```

Example layout:

```text
$WORKSPACE_ROOT/
  flutter_porthole/
    <slice-purpose>/
  zig-porthole-core/
    <slice-purpose>/
  zig-porthole-peer/
    <slice-purpose>/
```

Create workspaces from the nested repo that owns the work:

```bash
cd /home/techsaint/files/src/social_media/social_media_implemenation/<nested-repo>
mkdir -p "$WORKSPACE_ROOT/<repo-name>"
jj workspace add "$WORKSPACE_ROOT/<repo-name>/<slice-purpose>"
```

> **Nested-repo-under-a-root-tracked-tree trap (learned 2026-07-11, run 20260711T0110Z-v64spn).** A nested repo
> that lives *inside* a root-tracked tree (e.g. `dotnet/iroh_net`, which has its own `.git` under the root-tracked
> `dotnet/`) is **NOT** isolated by a root workspace — the root workspace symlinks it to the SHARED tree, and edits
> there are either refused or leak to other sessions. Materialize such a nested repo as its **own** workspace, the
> same as any other repo in the set. If a plan's `repo_set` lists both the parent (`dotnet/`) and a nested child
> (`dotnet/iroh_net`), isolate the child explicitly. Also confirm any cross-repo path a test reads (e.g.
> `platform_interop` vectors consumed by .NET tests) resolves inside the isolated layout, not via a shared symlink.
>
> **MSBuild ref-assembly race with symlinked dual .NET trees (learned 2026-07-11, run 20260711T0344Z-v64sp2).** When
> the root-tracked `dotnet/` and `dotnet_service` are materialized as symlinked dual trees, parallel MSBuild reference-
> assembly generation can race and fail the build. Setting `ProduceReferenceAssembly=false` (or building single-path /
> non-parallel) unblocked `Community.Tests`. If a .NET build in an isolated workspace fails with ref-assembly / restore
> errors rather than a code error, suspect the symlinked-tree layout, not the diff.
>
> **Cross-impl INTEGRATION gates are fresh-checkout gates, not in-workspace gates (learned 2026-07-11, run 3).** The
> isolation that makes *editing* safe (symlinked dual-trees) is exactly what breaks a cross-impl *integration* harness
> (e.g. `platform_interop/ci/run-all.sh`), whose sub-harnesses need a real multi-repo layout to resolve deps
> (`PORTHOLE_DOTNET_ROOT`, sibling test projects, native `.so` on the flutter test path). An in-workspace run of such
> a gate is dominated by layout artifacts (`exit 143 / path`, xunit assembly-load, missing sibling-test paths) and is
> NOT a valid gate result. Run the integration gate on a **fresh checkout of the full repo set at canonical tips with
> the workspace changes applied**, in the true relative layout — that is the authoritative promote gate (it is also
> what `.agent-rules/plan-amendment.md §3b` verify-on-fresh-checkout means for these gates). Unit/build gates per repo
> are fine in-workspace; whole-suite cross-impl aggregates are not.

Before scripting workspace creation, run:

```bash
jj workspace add --help
```

`jj` flags vary by installed version; verify the local CLI before relying on a
memorized command shape.

Keep repo and slice directory names human-readable. The `PLAN_RUN_ID` provides
collision resistance; do not add opaque GUIDs at every directory level unless a
tool requires it.

## Run Files (Claim Tickets)

For plan-driven work, **claim the run in root at start** by writing
`plans/<plan>/runs/<PLAN_RUN_ID>.md` (filename == `PLAN_RUN_ID`). This is the one
sanctioned root write that makes an isolated run discoverable from a cold
checkout — execution stays in the workspace; the claim is published to root.

> This is not optional for workspace-isolated runs: it is **slice-exit gate
> condition (e)** in `.agent-rules/plan-amendment.md §3`. This section is the
> *mechanism*; the *requirement* is enforced there.

```bash
# at run start, before working in the workspace:
cp docs/llm/skills/porthole-jj/run-file-template.md \
   plans/<plan>/runs/<PLAN_RUN_ID>.md
# fill in plan/run id/started/session/workspace path; status: in-progress
# mirror the one-line purpose into the workspace description:
jj desc -m "plan <plan> run <PLAN_RUN_ID> — <purpose>; see plans/<plan>/runs/<PLAN_RUN_ID>.md"
```

- **Liveness is derived** (`jj op log` recency in the workspace), not a hand-kept
  heartbeat — the ticket records `started`, jj says alive/dead.
- Flip `status` to `passed` / `abandoned` / `promoted` / `rejected` at exit.
- The ticket is the home for the Shared Branch Visibility block (below) and acts
  as an **advisory promotion lock** (a second run promoting to the same shared
  branch sees the in-progress claim).
- Reconcile with a two-way sweep: `runs/*.md` `in-progress` + no jj activity =
  candidate orphan; live workspace + no ticket = undocumented run.

Template + rationale: `run-file-template.md` (this dir),
`docs/decisions/2026-06-19-plan-run-files-and-claim-tickets.md`, and
`plans/README.md` ("Run files").

## Promotion And Shared Branch Visibility

Workspace commits are not the same as promoted shared-branch commits. For
multi-slice plan-driven work, do not export a workspace change to a shared nested
repo branch unless the active plan includes an explicit promotion/export phase
or the user/coordinating session approves the export.

Before any export or branch movement, report:

```text
Shared branch visibility:
- repo:
- branch:
- branch changed: yes/no
- old HEAD:
- new HEAD:
- visible to other sessions: yes/no
- export/promotion approval: not requested / requested / approved by <who> at <when>
```

Expected state for experiment/workspace slices:

```text
branch changed: no
visible to other sessions: no
export/promotion approval: not requested
```

If a plan is a speculative dev test or may be rejected before MVP, keep
implementation code in plan-run workspaces or backup refs until an explicit
promotion decision.

## Safety Rules

- Do not run destructive Git commands (`git reset --hard`, `git checkout --`) in
  a `jj`-managed repo unless explicitly requested.
- Avoid interactive `jj` commands in automated sessions. Do not use `jj split`,
  `jj resolve`, `jj squash` without explicit revision targets, or flags that
  open an editor. Prefer `jj desc -m "..."` or an explicit non-interactive
  command shape.
- Use `jj undo` for accidental `jj` operations.
- Use `jj diff`, `jj status`, `jj log`, and `jj op log` to understand state.
- If a `jj` change should become a Git-visible commit/bookmark, state the export
  step, get the required approval, and verify with both `jj status` and
  `git status`.
- If abandoning a `jj` experiment, report exactly what was abandoned and confirm
  no uncommitted nested repo work remains.
- After conflict resolution, check for `.jj-*` marker files in Git status. If
  marker files were accidentally staged or left as ghost index entries, clean
  only those marker entries and rerun `git status` plus `jj status`.

## Documentation

For repo changes involving the `jj` pilot:

- Update `docs/llm/skills/` if skill instructions change.
- File factual work in `docs/changelog/entries/`.
- File reusable experiment learning in `docs/notes/<area>/<topic>/`.
- Keep `.agents/skills` symlinks synced with `scripts/sync_repo_skills.sh`.

## When To Stop

Stop and ask before:

- converting the root repo to `jj`,
- replacing existing Git branch/worktree guidance,
- changing release/smoke scripts to require `jj`,
- making `jj` mandatory for contributors.
