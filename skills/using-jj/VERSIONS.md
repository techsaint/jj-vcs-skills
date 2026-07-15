# Version gates (jj CLI lines)

This skill pack’s **command recipes** are baseline-accurate for **jj 0.40.0**.
Each newer line adds **gates**: behavior or flags that differ from 0.40. Match
gates to the user’s `jj --version`.

| CLI line | Skill status |
|---|---|
| **0.40.0** | Baseline recipes |
| **0.41.0** | Gates below |
| 0.42.0+ | Not gated in this pack yet |

Official docs: https://www.jj-vcs.dev/v0.40.0/ · https://www.jj-vcs.dev/v0.41.0/

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

Prefer this for background status/log probes when on **≥ 0.41**.

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

### Still stub / not 0.41-only

- **`jj run`**: help may exist as a **stub** on 0.40–0.42; real behavior is a
  **0.43+** topic — do not teach as production workflow yet.
- Core recipes (`jj git init`, `jj squash --from/--into`, `jj bookmark`,
  `jj undo`, `bookmarks()`) remain as in the 0.40 baseline.
