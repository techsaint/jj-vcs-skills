# Version notes (jj CLI)

This pack is written for **jj 0.44.0**. Official docs:
https://www.jj-vcs.dev/v0.44.0/

Check `jj --version`. On **0.44**, follow the guides plus **Current line**
below. On an **older** CLI, use the matching section (or install tag
`using-jj/jj-<that-version>` for a snapshot that does not mention later lines).

Older official manuals: https://www.jj-vcs.dev/v0.43.0/ ·
https://www.jj-vcs.dev/v0.42.0/ · https://www.jj-vcs.dev/v0.41.0/ ·
https://www.jj-vcs.dev/v0.40.0/

---

## Current line — 0.44.0

What 0.44 adds or hardens vs **0.43** (already in the recipes unless noted):

### Tags fetch and push like bookmarks

Tracked tags are fetched and pushed by default. `jj git fetch` imports
remote tags as `<tag>@<remote>` and tracks same-name local tags (first
fetch after upgrade re-fetches tags to set tracking). Git `tagOpt` is
ignored — use `remotes.<name>.fetch-tags` (e.g. `'~*'` to disable).

```bash
jj tag track 'v1.0@origin'
jj tag untrack 'v1.0@origin'
jj git push --all          # bookmarks and tags
jj git push --allow-conflicts
```

`jj git clone` no longer accepts `--fetch-tags=all|none|included` (removed
in 0.44). Use `--tag=PATTERN` to limit which tags are fetched.

### `jj file search` prints matching lines

Each match is a line, prefixed by the file path. `--name-only` restores
path-only output. `-n` / `--line-number` prefixes the 1-based line number.
`--pattern` still defaults to **regex:** unless you set a kind.

### `jj run` order and flags

Revisions start **oldest to newest**. Start order is guaranteed even with
`-j` / `--jobs` > 1. New flags: `--passthrough` (child stdout/stderr to
the terminal), `--ignore-changes` (do not amend even if the WC changed),
`--ignore-errors` (keep going after a nonzero child exit).

```bash
jj run --passthrough -- cargo test
jj run --ignore-errors -- cargo check
```

On **0.42 and older**, `jj run` is a stub — do not use these recipes there.

### Revsets

| Change | Do instead |
|---|---|
| `merge_point(x)` **added** | Common descendant(s) of commits in `x` (counterpart of `fork_point`) |
| `builtin_log()` **added** | Built-in default `jj log` revset; `revsets.log` defaults to it |

Still true on 0.44 (from earlier lines): real `jj run`; `jj show --reversed`
and multi-rev `show`; `forks()`; do not use removed git-head / git-refs
revsets or `refs/heads/…` symbols; `--no-integrate-operation`; bulk
`jj git push` may **skip** ineligible bookmarks; do not use removed 0.42
flags (`--allow-new`, describe/commit author flags, old git config keys).

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
