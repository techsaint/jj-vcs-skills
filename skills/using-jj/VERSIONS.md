# Version notes (jj CLI)

This pack is written for **jj 0.45.1**. Official docs:
https://www.jj-vcs.dev/v0.45.1/

Check `jj --version`. On **0.45.1**, follow the guides plus **Current line**
below. On an **older** CLI, use the matching section (or install tag
`using-jj/jj-<that-version>` for a snapshot that does not mention later lines).

Older official manuals: https://www.jj-vcs.dev/v0.45.0/ ·
https://www.jj-vcs.dev/v0.44.0/ ·
https://www.jj-vcs.dev/v0.43.0/ · https://www.jj-vcs.dev/v0.42.0/ ·
https://www.jj-vcs.dev/v0.41.0/ · https://www.jj-vcs.dev/v0.40.0/

---

## Current line — 0.45.1

**Patch on 0.45.0.** Agent-facing commands and flags are the same (`jj
converge`, config `--file` / first `--user` file, `jj run` stops on a
failed child). Official 0.45.1 notes are crate-publish / `cargo install`
and SHA-256 Git signature storage (`gpgsig-sha256`). Use the 0.45 recipes
below.

What 0.45.0 added vs **0.44** (already in the recipes unless noted):

### `jj converge`

Resolves **divergent changes** (same change ID, more than one visible
revision). Groups the search revset by change ID, tries heuristics, and
replaces the divergent revisions with one solution. Descendants rebase;
local bookmarks move to the solution. Review with `jj op show -p` /
`jj evolog`; `jj undo` if the result is wrong.

```bash
jj converge --no-interactive
jj converge -r 'visible_heads()' --no-interactive
```

`--no-interactive` prints a warning and exits without changing the repo
if prompting would be required (prefer this in agents). Default search
is `revsets.converge` if you omit `-r`.

On **0.44 and older**, `jj converge` does not exist.

### Config `--user` / `--file`

`jj config edit|set|unset --user` writes the **first loaded** user
config file (`~/.config/jj/config.toml` or the first `conf.d/` file),
not an interactive picker. Use `--file PATH` to target a specific file.

### `jj run` stops on failure

Default: if a child exits nonzero, remaining revisions are **not** run.
`--ignore-errors` still continues (same flag as 0.44).

### Other 0.45 notes

- Default `immutable_heads()` includes `untracked_remote_tags()`.
- Non-colocated `jj git import` no longer imports a detached Git HEAD.

Still true on 0.45 (from earlier lines): tracked tags; clone `--tag`
(not `--fetch-tags`); file-search line output; `jj run` oldest-first
plus `--passthrough` / `--ignore-changes`; `merge_point(x)`; real
`jj run` (0.43+); do not use removed git-head / git-refs revsets or
`refs/heads/…` symbols.

---

## If you are on 0.45.0

Same recipes as 0.45.1. No `jj converge` / config / `jj run` differences.

---

## If you are on 0.44.0

Apply the 0.43 notes below, plus:

- Tracked tags: `jj tag track`/`untrack`; `jj git push --all` includes
  tags; `--allow-conflicts`. Clone: `--tag=PATTERN`, not `--fetch-tags`.
- File search prints matching lines (`--name-only` for paths only).
- `jj run` oldest-first; `--passthrough` / `--ignore-changes` /
  `--ignore-errors`.
- `merge_point(x)`; `builtin_log()`.
- **No** `jj converge`. Config `--user` may still prompt when multiple
  user files exist. `jj run` without `--ignore-errors` may have kept
  going after a failed child (0.45 stops).

---

## If you are on 0.43.0

Apply the 0.42 notes below, plus:

- **`jj run` is real** (isolated WC per revision, amend, descendant rebase).
  No `--passthrough` / `--ignore-changes` / `--ignore-errors`.
- `jj show --reversed`; `forks()` revset.
- `git_head()` / `git_refs()` **removed**; `refs/heads/…` symbols do not
  resolve; `jj bookmark track`/`untrack` take `<bookmark>@<remote>` only.
- Fetch rebases descendants of change-ID-rewritten revisions more completely
  than 0.42.
- **No** `jj tag track`/`untrack`. Clone still has `--fetch-tags=…` (removed
  on 0.44). File search prints **paths**, not every matching line.
- `jj git push --all` is bookmarks only (0.44 also pushes tags).

---

## If you are on 0.42.0

Apply the 0.41 notes below, plus:

- `jj show` takes `[REVSETS]...` (multi-rev).
- `jj util backend name` prints the commit backend (usually `git`).
- Fetch may generate evolution from change IDs and rebase descendants.
- Removed (hard-fail): `jj git push --allow-new`; `jj describe`/`commit`
  `--author` / `--reset-author` / `--no-edit` / `--edit`; `jj metaedit
  --update-committer-timestamp`; config `git.auto-local-bookmark` /
  `git.push-new-bookmarks`.
- `jj describe --editor` and `jj new --no-edit` still exist (different commands).
- **`jj run` is still a stub** — not a working workflow.

---

## If you are on 0.41.0

- `--no-integrate-operation` exists (absent on 0.40).
- `jj file search --pattern` omitted kind defaults to **regex:**, not glob.
  Use `regex:…` or `glob:…` explicitly.
- `jj git push --all` / `--tracked` / `-r` may **skip** private/conflict
  bookmarks (exit 0 is not proof everything pushed).
- Clone bookmark/tag patterns live in **jj repo settings**, not only
  `.git/config`.
- Prefer `Operation.attributes()` over deprecated `Operation.tags()`.
- No multi-rev `jj show`, no real `jj run`, no `jj util backend name`.

---

## If you are on 0.40.0

Guides use modern surfaces (`jj git init`, `jj bookmark`, `jj squash
--from/--into`, `jj undo`, `bookmarks()`). Do **not** use 0.41+ flags
(`--no-integrate-operation`) or 0.43+ `jj run`. File-search pattern default
may not be regex. Bulk push may still hard-fail instead of skipping.
