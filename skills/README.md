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

| Skill | Description |
|---|---|
| [using-jj](./using-jj/) | General Jujutsu (jj) skill; CLI version gates |

## using-jj

- Pack: [using-jj/](./using-jj/)
- Skills file: [using-jj/SKILL.md](./using-jj/SKILL.md)
- Version gates: [using-jj/VERSIONS.md](./using-jj/VERSIONS.md)
- Repo overview: [../README.md](../README.md)
- Version pins: GitHub branches and tags (see root [README](../README.md))

## Install examples

```bash
# Symlink into a discovery root
ln -s "$(pwd)/using-jj" /path/to/.agents/skills/using-jj

# Or set the tool’s skills directory to this folder (multi-skill root)
# skills_root = /path/to/this/repo/skills
```
