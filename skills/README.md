# Skills (install root)

Point agent skill discovery at **this directory** (`skills/`), or copy/symlink
individual skill folders into your tool’s skills path.

Install-shaped layout:

```text
skills/                      ← discovery root
  README.md                  ← this catalog
  using-jj/
    SKILL.md                 ← agent entry (required)
    README.md                ← humans
    VERSIONS.md
    …
  porthole-jj/
    SKILL.md
    …
```

| Skill | Portable? | Description |
|---|---|---|
| [using-jj](./using-jj/) | **Yes** — community | General Jujutsu (jj) skill; CLI version gates |
| [porthole-jj](./porthole-jj/) | **No** — project pilot | Nested-repo / plan-run jj pilot patterns |

## using-jj (public product)

- Humans: [using-jj/README.md](./using-jj/README.md)
- Agents: [using-jj/SKILL.md](./using-jj/SKILL.md)
- Releases (tags in this repo): `using-jj/jj-0.40.0`, `using-jj/jj-0.41.0`, …

## Install examples

```bash
# Symlink into a Codex-style discovery root
ln -s "$(pwd)/using-jj" /path/to/.agents/skills/using-jj

# Or set the tool’s skills directory to this folder (multi-skill root)
# skills_root = /path/to/this/repo/skills
```

Parent monorepo maintainers: product path is `src/skills/using-jj/`;
frozen consumer install remains `docs/llm/skills/` until product M5.
