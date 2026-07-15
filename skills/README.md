# Skills (install root)

Point skill discovery at **this directory** (`skills/`), or copy/symlink
individual skill folders into your tool’s skills path.

```text
skills/                      ← discovery root
  README.md                  ← this catalog
  using-jj/
    SKILL.md                 ← skills file (required)
    VERSIONS.md
    …
```

| Skill | Portable? | Description |
|---|---|---|
| [using-jj](./using-jj/) | **Yes** — community | General Jujutsu (jj) skill; CLI version gates |

`porthole-jj` (Porthole monorepo pilot) was **removed** — not portable, not
maintained for 0.40+ gates. Isolation/run-file ideas belong in project docs or
a future generic scaffold, not this public catalog.

## using-jj (public product)

- Pack: [using-jj/](./using-jj/)
- Skills file: [using-jj/SKILL.md](./using-jj/SKILL.md)
- Version gates: [using-jj/VERSIONS.md](./using-jj/VERSIONS.md)
- Repo overview: [../README.md](../README.md)
- Releases (tags in this repo): `using-jj/jj-0.40.0`, `using-jj/jj-0.41.0`, …

## Install examples

```bash
# Symlink into a discovery root
ln -s "$(pwd)/using-jj" /path/to/.agents/skills/using-jj

# Or set the tool’s skills directory to this folder (multi-skill root)
# skills_root = /path/to/this/repo/skills
```

Parent monorepo: product path is `src/skills/using-jj/`;
frozen consumer install remains `docs/llm/skills/` until product M5.
