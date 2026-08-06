# Version gates (jj CLI lines)

This skill pack supports **jj through 0.43.0**.

- **Baseline recipes** (guides + `SKILL.md` workflows): written for **jj 0.40.0**.
- **0.41.0 / 0.42.0 / 0.43.0 gates** (this file): differences from 0.40 that apply
  when the user’s `jj --version` matches that line.

Match gates to the installed CLI. Official docs:
https://www.jj-vcs.dev/v0.40.0/ · https://www.jj-vcs.dev/v0.41.0/ ·
https://www.jj-vcs.dev/v0.42.0/ · https://www.jj-vcs.dev/v0.43.0/

| CLI line | Skill status |
|---|---|
| **0.40.0** | Baseline recipes |
| **0.41.0** | Gates below |
| **0.42.0** | Gates below (includes 0.41) |
| **0.43.0** | Gates below (includes 0.41–0.42) |

---

## Since 0.41.0

### Automation: `--no-integrate-operation`

Global flag (absent on 0.40). Run a command **without** integrating an operation
into the repo / working-copy snapshot path — useful when tools would otherwise
create noise ops.

```bash
jj --no-integrate-operation status
# still performs real work for some network commands (e.g. push may still push);
# read `jj --help` for the flag’s caveats on your binary
```

Prefer this for background status/log probes when on **0.41+**.

### `jj file search --pattern` default is **regex:**

On 0.41, omitted kind defaults to **regex**, not glob (breaking vs earlier
expectation). Use explicit kinds:

```bash
jj file search --pattern 'regex:foo.*bar'
jj file search --pattern 'glob:*foo*'    # whole-line glob; often need *…*
```

If “shell glob” is assumed without a kind, matches will be wrong on 0.41+.

### `jj git push --all` / `--tracked` / `-r` may **skip** ineligible bookmarks

On 0.41, push no longer hard-fails the whole command when some revisions are
**private** or have **conflicts**. Ineligible bookmarks are **skipped**.

**Rule:** exit code 0 is not proof that every intended bookmark pushed.
After bulk push, verify with `jj bookmark list` / remote state / command output.
Use `--allow-private` when private commits must be pushed (see `jj git push --help`).

### `jj git clone` bookmark/tag patterns → jj repo settings

Patterns from clone are stored in **jj’s repo settings**, not only `.git/config`.
Do not teach “edit `.git/config` refspecs” as the sole place clone filters live
on 0.41+.

### Templates: `Operation.tags()` deprecated

Prefer **`Operation.attributes()`** on 0.41+. (This pack rarely uses op template
fields; if you add them, use `attributes`.)

### Unchanged from 0.40 baseline

Core recipes (`jj git init`, `jj squash --from/--into`, `jj bookmark`,
`jj undo`, `bookmarks()`) remain as in the 0.40 baseline guides.

---

## Since 0.42.0

Apply **all 0.41 gates above**, plus:

### `jj show` accepts multiple revisions

On 0.42, `jj show` takes `[REVSETS]...` and prints each revision in turn
(closer to `git show`).

```bash
jj show @ @-          # two revisions, one after the other
jj show abc123 def456
```

Single-rev `jj show <rev>` still works.

### `jj util backend name`

Prints the commit backend for the current repo (typically `git` for colocated
repos):

```bash
jj util backend name
```

### `jj git fetch` and change-ID evolution

On 0.42, fetch can generate evolution history from **change IDs**. When the
remote preserves change IDs, local **descendant** revisions may be **rebased**
onto rewritten parents after fetch.

**Rule:** after `jj git fetch` on shared stacks, re-check `jj log` / bookmark
targets — descendants may have moved, not only remote bookmark tips.

### Removed options (hard-fail if taught)

These deprecated options are **gone** on 0.42 (not merely warned):

| Removed | Do instead |
|---|---|
| `jj git push --allow-new` | Push the bookmark / `--change` / `--all` as appropriate; new bookmarks track on push when eligible — see `jj git push --help` |
| `jj describe --author` / `--reset-author` / `--no-edit` / `--edit` | Use current describe flags (`-m`, `--stdin`, `--editor`, …); author rewrite via current metaedit / config surfaces on your binary |
| `jj commit --author` / `--reset-author` | Same — check `jj commit --help` on 0.42 |
| `jj metaedit --update-committer-timestamp` | See `jj metaedit --help` for current options |
| Config `git.auto-local-bookmark` | Removed — do not set in setup recipes |
| Config `git.push-new-bookmarks` | Removed — do not set in setup recipes |

Do **not** confuse `jj describe --edit` (removed) with `jj describe --editor`
(still present), or with `jj new --no-edit` (still present — different command).

### Still a stub: `jj run`

`jj run` remains a **stub** on 0.42 (help exists; does not work for production
use). Do not teach it as a working workflow on this line. Real `jj run` is
**0.43+** (next section).

---

## Since 0.43.0

Apply **all 0.41 and 0.42 gates above**, plus:

### `jj run` is real (0.43+ only)

On 0.43, `jj run` checks out each selected revision in an **isolated working
copy**, runs a command, then **amends** that revision. Descendants rebase onto
the amended commits by default (`--restore-descendants` keeps descendant
*content*). Do **not** use this recipe on 0.40–0.42 (stub only).

```bash
jj run -- cargo check --all-features
jj run -- cargo fix
jj run -j 4 -- pre-commit run
jj run -r 'bookmarks()..@' -- cargo test
```

Hints from 0.43 help: use `--` before flags meant for the inner command;
`--root` runs from the working-copy root; `--clean` deletes reused workspaces
between invocations; env vars `JJ_CHANGE_ID`, `JJ_COMMIT_ID`, `JJ_WORKSPACE_ROOT`
are set.

### `jj show --reversed`

On 0.43, `jj show` accepts `--reversed` (in addition to multi-rev from 0.42).

### Revsets / symbols removed or added

| Change | Do instead |
|---|---|
| `git_head()` / `git_refs()` **removed** | Do not use; they hard-fail. Prefer current bookmark / git-tracking revsets on your binary (`jj help -k revsets`) |
| Git-like `refs/heads/main` **no longer resolves** | Use bookmark/tag `main` or `main@origin` |
| `jj bookmark track` / `untrack` no `<kind>:` patterns | Use `<bookmark>@<remote>` only |
| `forks()` **added** | Commits with more than one child — useful for merge/stack queries |

### `jj git fetch` stack rebase (0.43)

Fetch rebases descendants of change-ID-rewritten revisions more completely than
0.42 (including stacks with multiple bookmarked revisions). Immutable
descendants are not rebased. After fetch on a stack, re-check `jj log`.
