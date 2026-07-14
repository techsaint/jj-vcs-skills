# Skills source repo (nested)

This directory is a **separate jj/git repository** from the parent
`jj-skills` workbench. The parent gitignores `src/`.

## Layout (multi-skill / install-shaped)

```text
src/                         # this repo root
  README.md
  skills/                    # ← point agent skills discovery here
    README.md                # catalog
    using-jj/                # community skill pack
      SKILL.md
      README.md
      …
```

| Path | Role |
|---|---|
| [`skills/using-jj/`](./skills/using-jj/) | Portable community skill for general jj |
| [`skills/README.md`](./skills/README.md) | Catalog + install notes |

## Releases (jj tags)

```text
using-jj/jj-0.40.0   # baseline M3
using-jj/jj-0.41.0   # + version gates M3 (main tip)
```

```bash
jj tag list
jj new using-jj/jj-0.41.0
```

Convention: parent doc `docs/notes/releases/skill-version-tags.md`.

## Validation (from monorepo root)

```bash
../tools/install/jj/bin/jj-0.40.0 --version
python3 ../scripts/check_using_jj_skill.py --version 0.41.0
# default skill-dir: src/skills/using-jj
```
