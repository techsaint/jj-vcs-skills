# Version notes (jj CLI)

This pack is written for **jj 0.43.0**. Official docs:
https://www.jj-vcs.dev/v0.43.0/

Check `jj --version`. On **0.43**, follow the guides plus **Current line**
below. On an **older** CLI, use the matching section (or install tag
`using-jj/jj-<that-version>` for a snapshot that does not mention later lines).

Older official manuals: https://www.jj-vcs.dev/v0.42.0/ ·
https://www.jj-vcs.dev/v0.41.0/ · https://www.jj-vcs.dev/v0.40.0/

---

## Current line — 0.43.0

What 0.43 adds or hardens vs **0.42** (already in the recipes unless noted):

### `jj run` is real

Checks out each selected revision in an **isolated working copy**, runs a
command, then **amends** that revision. Descendants rebase onto the amended
commits by default (`--restore-descendants` keeps descendant *content*).

```bash
jj run -- cargo check --all-features
jj run -- cargo fix
jj run -j 4 -- pre-commit run
jj run -r 'bookmarks()..@' -- cargo test
```

Use `--` before flags meant for the inner command; `--root` runs from the
working-copy root; `--clean` deletes reused workspaces between invocations.
Env vars: `JJ_CHANGE_ID`, `JJ_COMMIT_ID`, `JJ_WORKSPACE_ROOT`.

On **0.42 and older**, `jj run` is a stub — do not use this recipe there.

### `jj show --reversed`

`jj show` accepts `--reversed` (and still accepts multiple revisions).

### Revsets / symbols

| Change | Do instead |
|---|---|
| `git_head()` / `git_refs()` **removed** | Do not use; they hard-fail. Prefer current bookmark / git-tracking revsets (`jj help -k revsets`) |
| Git-like `refs/heads/main` **no longer resolves** | Use bookmark/tag `main` or `main@origin` |
| `jj bookmark track` / `untrack` no `<kind>:` patterns | Use `<bookmark>@<remote>` only |
| `forks()` **added** | Commits with more than one child |

### `jj git fetch` stack rebase

Fetch rebases descendants of change-ID-rewritten revisions more completely than
0.42 (including stacks with multiple bookmarked revisions). Immutable
descendants are not rebased. After fetch on a stack, re-check `jj log`.

Still true on 0.43 (from earlier lines): `--no-integrate-operation`; file-search
`--pattern` defaults to **regex:**; bulk `jj git push` may **skip** ineligible
bookmarks; `jj show` takes multiple revisions; do not use removed 0.42 flags
(`--allow-new`, describe/commit author flags, old git config keys).

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
(`--no-integrate-operation`) or 0.43 `jj run`. File-search pattern default may
not be regex. Bulk push may still hard-fail instead of skipping.
