# using-jj — community skill pack for Jujutsu (jj)

Portable **LLM skill** (and human reference) for working with
[Jujutsu](https://github.com/jj-vcs/jj): mental model, setup, git interop,
workflows, stacked PRs, and **CLI-line version gates**.

This directory is the **product pack** under development in the nested `src/`
repo. It is versioned against **jj CLI releases** (not an arbitrary skill
semver alone).

| jj CLI line | Pack status | Release tag (in `src/` repo) |
|---|---|---|
| **0.40.0** | Baseline recipes — maturity **M3** | `using-jj/jj-0.40.0` |
| **0.41.0** | Gates on top of baseline — maturity **M3** | `using-jj/jj-0.41.0` |
| 0.42.0+ | Not finished in this pack yet | — |

Official docs for those lines:  
https://www.jj-vcs.dev/v0.40.0/ · https://www.jj-vcs.dev/v0.41.0/

---

## Quick start

| You are… | Open |
|---|---|
| An **agent** loading the skill | [`SKILL.md`](./SKILL.md) (TRIGGER + index) |
| Checking **which jj version** rules apply | [`VERSIONS.md`](./VERSIONS.md) |
| A **human** browsing the pack | This README, then the table below |

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

---

## What’s in the pack

| File | Role |
|---|---|
| [`SKILL.md`](./SKILL.md) | Agent entry: target versions, TRIGGER, index, common workflows |
| [`VERSIONS.md`](./VERSIONS.md) | **Version gates** (0.40 baseline + 0.41+ deltas) |
| [`jj-mental-model.md`](./jj-mental-model.md) | How jj thinks (change IDs, op log, revsets, …) |
| [`jj-setup-guide.md`](./jj-setup-guide.md) | Install, config, colocate, migration |
| [`jj-git-command-reference.md`](./jj-git-command-reference.md) | git → jj command map |
| [`jj-powerful-features.md`](./jj-powerful-features.md) | What git can’t do easily |
| [`jujutsu-workflows.md`](./jujutsu-workflows.md) | Daily / collab / history / large-repo workflows |
| [`jj-stacked-prs-guide.md`](./jj-stacked-prs-guide.md) | Bookmark-based stacked PRs |
| [`jj-convert-to-stacked-prs.md`](./jj-convert-to-stacked-prs.md) | Reorganize an existing branch into a stack |

---

## Version policy (read this)

1. **Baseline = 0.40.0** — recipes in the guides are written for that CLI.
2. **Newer lines** — open [`VERSIONS.md`](./VERSIONS.md) and apply the section for
   your `jj --version` (e.g. 0.41 push-skip and regex file-search defaults).
3. **Do not** assume latest-only features (e.g. production `jj run`) work on 0.40/0.41.

If you maintain this pack inside the **jj-skills** monorepo:

- Validate with **repo-local binaries**, not PATH jj:
  `tools/install/jj/bin/jj-0.40.0` / `jj-0.41.0` (from monorepo root)
- Product maturity cards: `docs/product/features/jj-0.40.0.md`, `jj-0.41.0.md`
- Confirmation process: `plans/confirm-jj-version-features/README.md`
- Automated gate: `python3 scripts/check_using_jj_skill.py --version 0.41.0
  (default `--skill-dir src/skills/using-jj`)`

---

## Install / use as an agent skill

Copy or symlink this directory where your agent discovers skills (tool-specific).
Entry file is **`SKILL.md`** (YAML frontmatter + body).

Example layout for many agents:

```text
…/skills/using-jj/SKILL.md
…/skills/using-jj/*.md   # keep the pack together
```

This pack is **portable**: no private monorepo paths required for day-to-day
jj help. Monorepo-only paths above are for *maintainers* of jj-skills.

---

## Releases

In the nested **`src/`** jj/git repo:

```bash
jj tag list
# using-jj/jj-0.40.0
# using-jj/jj-0.41.0

jj new using-jj/jj-0.40.0   # work from the 0.40 snapshot
```

Tag naming and dual-repo layout: parent doc  
`docs/notes/releases/skill-version-tags.md` (from monorepo root).

---

## Why this exists

Jujutsu is **underserved in the LLM skill ecosystem**. Outdated community packs
still teach removed subcommands and revsets. This pack is maintained against real
release binaries and version-pinned docs so agents get **checkable** guidance
per CLI line.

---

## License / upstream

- Skill content: same policy as the enclosing `src/` / project repository.
- jj itself: https://github.com/jj-vcs/jj  
