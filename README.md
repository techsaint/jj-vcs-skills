# Skills source repo (nested)

This directory is a **separate jj/git repository** from the parent
`jj-skills` tree. The parent gitignores `src/` so skill history lives here.

## Trees

| Path | Role |
|---|---|
| `using-jj/` | Portable community skill for general jj — start at [`using-jj/README.md`](./using-jj/README.md) |
| `porthole-jj/` | Project-local pilot patterns |

## Releases (jj tags)

Skill packs that target a given **jj CLI** line are pinned with tags:

```text
using-jj/jj-0.40.0   # baseline M3
using-jj/jj-0.41.0   # + version gates M3 (main tip)
…
```

List / use:

```bash
jj tag list
jj new using-jj/jj-0.40.0
```

Full dual-repo convention (parent tags, `main`, cut process):

`../docs/notes/releases/skill-version-tags.md`

## Validation

Use parent repo-local binaries, not PATH `jj`:

```bash
../tools/install/jj/bin/jj-0.40.0 --version
```
