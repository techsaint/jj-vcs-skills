# jj agent skills

**Community-oriented LLM skills for [Jujutsu (jj)](https://github.com/jj-vcs/jj).**

The LLM skill ecosystem underserves jj: packs still teach removed or
version-wrong commands. This repository maintains **checkable** skill content
against real release binaries and version-pinned docs, line by line (0.40, 0.41, …).

| | |
|---|---|
| **Product skill** | [`skills/using-jj/`](./skills/using-jj/) |
| **Agent entry** | [`skills/using-jj/SKILL.md`](./skills/using-jj/SKILL.md) |
| **Humans (pack)** | [`skills/using-jj/README.md`](./skills/using-jj/README.md) |
| **CLI version gates** | [`skills/using-jj/VERSIONS.md`](./skills/using-jj/VERSIONS.md) |

---

## Status (jj CLI lines)

| jj CLI | Skill gates | Tag (this repo) |
|---|---|---|
| **0.40.0** | Baseline recipes (M3) | `using-jj/jj-0.40.0` |
| **0.41.0** | + version gates (M3) | `using-jj/jj-0.41.0` (tip) |
| 0.42.0 | Not finished yet | — |
| 0.43.0 | Not finished yet | — |

Official docs: [v0.40.0](https://www.jj-vcs.dev/v0.40.0/) · [v0.41.0](https://www.jj-vcs.dev/v0.41.0/)

---

## Install (agent skills root)

This repo is **multi-skill / install-shaped**. Point your agent’s skills
discovery at the **`skills/`** directory (or copy a single skill folder into
your existing skills root).

```text
skills/                 ← set as skills root (or clone path + /skills)
  using-jj/
    SKILL.md            ← agents load this
    README.md
    VERSIONS.md
    …
```

### Examples

```bash
# Clone (when published)
git clone https://github.com/techsaint/jj-vcs-skills.git
# or: jj git clone …

# Codex-style: symlink one skill
ln -s "$(pwd)/skills/using-jj" /path/to/.agents/skills/using-jj

# Or point the tool at the whole multi-skill root:
#   skills_directory = /path/to/this-repo/skills
```

Exact config keys depend on the agent (Claude Code, Cursor, Codex, etc.).
What matters: the tool sees a directory named `using-jj` that contains
**`SKILL.md`**.

---

## Layout

```text
.                          # this repository root (GitHub front door)
├── README.md              # ← you are here
└── skills/                # agent discovery root
    ├── README.md          # short catalog
    └── using-jj/          # portable community skill
        ├── SKILL.md
        ├── README.md
        ├── VERSIONS.md
        └── …
```

---

## Releases

Tags pin skill content to a **jj CLI** line:

```bash
jj tag list
# using-jj/jj-0.40.0
# using-jj/jj-0.41.0

jj new using-jj/jj-0.40.0    # work from the 0.40 snapshot
```

---

## Maintainers (jj-skills monorepo)

If this tree lives as a **nested** repo under a parent workbench
(`jj-skills/src/` gitignored from the parent):

| Concern | Where |
|---|---|
| Product desk / maturity | Parent `docs/product/features/jj-0.4x.0.md` |
| Confirmation process | Parent `plans/confirm-jj-version-features/` |
| CLI checker | Parent `scripts/check_using_jj_skill.py` |
| Repo-local jj binaries | Parent `tools/install/jj/bin/jj-<ver>` |
| Tags / dual-repo notes | Parent `docs/notes/releases/skill-version-tags.md` |

From the monorepo root:

```bash
python3 scripts/check_using_jj_skill.py --version 0.41.0
# default --skill-dir src/skills/using-jj
```

---

## License

TODO before public release — add a `LICENSE` file (e.g. MIT / Apache-2.0).

---

## Related

- Upstream jj: https://github.com/jj-vcs/jj  
- Versioned docs: https://www.jj-vcs.dev/  
