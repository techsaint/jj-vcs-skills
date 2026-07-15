# jj-vcs skills

**Community-oriented skills for [Jujutsu (jj)](https://github.com/jj-vcs/jj).**

The LLM skill ecosystem underserves jj: packs still teach removed or
version-wrong commands. This repository maintains **checkable** skill content
against real release binaries and version-pinned docs, line by line (0.40, 0.41, …).

Content and layout borrow from the
[Superpowers](https://github.com/obra/superpowers) skill ecosystem (and related
community packs), reworked and re-verified for modern jj CLI lines.

| | |
|---|---|
| **Skill pack** | [`skills/using-jj/`](./skills/using-jj/) |
| **Skills file** | [`skills/using-jj/SKILL.md`](./skills/using-jj/SKILL.md) |
| **CLI version gates** | [`skills/using-jj/VERSIONS.md`](./skills/using-jj/VERSIONS.md) |

Official docs: [v0.40.0](https://www.jj-vcs.dev/v0.40.0/) · [v0.41.0](https://www.jj-vcs.dev/v0.41.0/)

---

## Status (jj CLI lines)

| jj CLI | Skill gates | Tag (this repo) |
|---|---|---|
| **0.40.0** | Baseline recipes | `using-jj/jj-0.40.0` |
| **0.41.0** | + version gates | `using-jj/jj-0.41.0` |
| 0.42.0 | Not finished yet | — |
| 0.43.0 | Not finished yet | — |

Which line is “current” is a **release** concern: use this repo’s default
branch, other branches, and tags on GitHub — not wording in this README.

---

## Quick start

**Core workflow (any supported line):**

```bash
jj status
# edit files (auto-tracked)
jj describe -m "what changed"
jj new
jj log
jj git push   # or: jj git push --change @
```

Baseline commands use modern surfaces: `jj git init`, `jj bookmark`,
`jj squash --from/--into`, top-level `jj undo`, revset `bookmarks()`.
(Avoid removed forms: bare `init`, `move`, `branch`, `op undo`, `branches()`.)

| Need | Open |
|---|---|
| Load the skill (TRIGGER + index) | [`skills/using-jj/SKILL.md`](./skills/using-jj/SKILL.md) |
| Which jj version rules apply | [`skills/using-jj/VERSIONS.md`](./skills/using-jj/VERSIONS.md) |

---

## Install

This repo is **multi-skill / install-shaped**. Point skill discovery at the
**`skills/`** directory, or copy/symlink a single skill folder into an existing
skills root.

```text
skills/                 ← set as skills root (or clone path + /skills)
  using-jj/
    SKILL.md            ← skills file (required)
    VERSIONS.md
    …
```

```bash
git clone https://github.com/techsaint/jj-vcs-skills.git
# or: jj git clone …

# Symlink one skill into a discovery root
ln -s "$(pwd)/skills/using-jj" /path/to/.agents/skills/using-jj

# Or point the tool at the whole multi-skill root:
#   skills_directory = /path/to/this-repo/skills
```

Exact config keys depend on the host tool (Claude Code, Cursor, Codex, etc.).
What matters: the tool sees a directory named `using-jj` that contains
**`SKILL.md`**.

---

## What’s in the pack

| File | Role |
|---|---|
| [`SKILL.md`](./skills/using-jj/SKILL.md) | Skills file: target versions, TRIGGER, index, common workflows |
| [`VERSIONS.md`](./skills/using-jj/VERSIONS.md) | **Version gates** (0.40 baseline + 0.41+ deltas) |
| [`jj-mental-model.md`](./skills/using-jj/jj-mental-model.md) | How jj thinks (change IDs, op log, revsets, …) |
| [`jj-setup-guide.md`](./skills/using-jj/jj-setup-guide.md) | Install, config, colocate, migration |
| [`jj-git-command-reference.md`](./skills/using-jj/jj-git-command-reference.md) | git → jj command map |
| [`jj-powerful-features.md`](./skills/using-jj/jj-powerful-features.md) | What git can’t do easily |
| [`jujutsu-workflows.md`](./skills/using-jj/jujutsu-workflows.md) | Daily / collab / history / large-repo workflows |
| [`jj-stacked-prs-guide.md`](./skills/using-jj/jj-stacked-prs-guide.md) | Bookmark-based stacked PRs |
| [`jj-convert-to-stacked-prs.md`](./skills/using-jj/jj-convert-to-stacked-prs.md) | Reorganize an existing branch into a stack |

---

## Version policy

1. **Baseline = 0.40.0** — recipes in the guides are written for that CLI.
2. **Newer lines** — open [`VERSIONS.md`](./skills/using-jj/VERSIONS.md) and apply the section for your `jj --version` (e.g. 0.41 push-skip and regex file-search defaults).
3. **Do not** assume latest-only features (e.g. production `jj run`) work on 0.40/0.41.

Pin a snapshot via GitHub **branches** and **tags** when you need a fixed CLI
line; the default branch is whatever this project currently publishes as HEAD.

---

## Layout

```text
.                          # this repository root (GitHub front door)
├── README.md              # ← you are here
└── skills/                # skill discovery root
    ├── README.md          # short catalog
    └── using-jj/          # portable community skill
        ├── SKILL.md       # skills file
        ├── VERSIONS.md
        └── …
```

---

## Why this exists

Jujutsu is underserved in the LLM skill ecosystem. Outdated community packs still
teach removed subcommands and revsets. This pack is maintained against real
release binaries and version-pinned docs so guidance is **checkable** per CLI line.

---

## Maintainers

TODO: add maintainer notes (how to verify skill content against a jj CLI line,
how tags/branches are cut, contribution expectations).

---

## License

[MIT](./LICENSE) — Copyright (c) 2026 Chris Butler.

---

## Related

- Upstream jj: https://github.com/jj-vcs/jj  
- Versioned docs: https://www.jj-vcs.dev/  
- Superpowers skills: https://github.com/obra/superpowers  
