---
name: using-jj
description: Use when working with Jujutsu (jj) version control - provides comprehensive guide on jj commands, workflows, setup, mental model, and powerful features that distinguish it from git
---

# Using Jujutsu (jj) Version Control

## Target version

**Baseline recipes: jj 0.40.0.** Core command surfaces (init/squash/bookmark/undo/…)
are accurate for **0.40.0**.

**Version gates through jj 0.41.0** are documented in
[VERSIONS.md](./VERSIONS.md) (e.g. `--no-integrate-operation`, file-search pattern
default **regex:**, push skip behavior). Apply that file when the user’s CLI is
**0.41.0** (or when unsure, check `jj --version`).

Do **not** assume 0.43-only features (e.g. real `jj run`) work on 0.40/0.41.

**Validate with the repo-local binary** (not system PATH `jj`):

```bash
./tools/install/jj/bin/jj-0.40.0 --version   # baseline
./tools/install/jj/bin/jj-0.41.0 --version   # with VERSIONS.md gates
# Docs: https://www.jj-vcs.dev/v0.40.0/  ·  https://www.jj-vcs.dev/v0.41.0/
```

Research / features: `docs/research/jj-feature-audit/v0.40.0/`,
`docs/research/jj-feature-audit/v0.41.0/`, `docs/product/features/jj-0.40.0.md`,
`docs/product/features/jj-0.41.0.md`.

## TRIGGER
Load when you need **general jj** command translation, mental model, or a feature not
covered by `porthole-jj` (the Porthole-specific layer). Pair with `porthole-jj` for any
plan-workspace work. SKIP if the task does not use jj.

## Overview

Jujutsu (jj) is a next-generation version control system that builds on git's object model but introduces powerful concepts that make version control more intuitive and forgiving. This skill provides comprehensive guidance on using jj effectively.

**Core Philosophy:**
- Working copy as a commit (no staging confusion)
- First-class conflicts (conflicts can be committed and resolved later)
- Operation log with true undo (every operation is reversible)
- Automatic rebase (descendants update automatically)
- Change IDs (track semantic changes across rewrites)
- Anonymous branches (branches are optional, not mandatory)

## When to Use This Skill

Use this skill when:
- Learning jj for the first time
- Migrating from git to jj
- Looking up jj command equivalents for git operations
- Trying to understand jj's mental model
- Setting up jj in your environment
- Learning advanced jj workflows
- Discovering what jj can do that git cannot

## Quick Start

### For Git Users
If you know git and want jj equivalents fast:
→ See: [jj-git-command-reference.md](./jj-git-command-reference.md) - Load this when you know what git command you want to do and need the jj equivalent. Essential reference for translating existing git knowledge to jj commands.

### For New Users
If you're learning version control concepts:
→ Start with: [jj-mental-model.md](./jj-mental-model.md) (Load this to understand jj's architecture and core concepts like change IDs, operation log, and revsets) → [jj-setup-guide.md](./jj-setup-guide.md) (Load this to install and configure jj) → [jujutsu-workflows.md](./jujutsu-workflows.md) (Load this to learn practical daily workflows)

### For Advanced Users
If you want to leverage jj's unique power:
→ See: [jj-powerful-features.md](./jj-powerful-features.md) (Load this to discover what jj can do that git cannot, like first-class conflicts and automatic rebasing) → [jujutsu-workflows.md](./jujutsu-workflows.md) (Load this for advanced workflows like stacked changes, history editing, and large repo strategies)

## Documentation Index

This skill includes seven comprehensive reference documents plus version gates:

### 0. [VERSIONS.md](./VERSIONS.md)
**CLI version gates (0.40 baseline + 0.41+ deltas)**

**Load when:** the user’s `jj --version` is not exactly the baseline, or the task
is automation / bulk push / file search patterns.

### 1. [jj-git-command-reference.md](./jj-git-command-reference.md) (23KB)
**Complete command mappings from git to jj**

**Load when:** You need to translate a specific git command to jj, or you're a git user learning jj command-by-command.

Coverage:
- Repository initialization and cloning
- Basic workflow (add, commit, status, diff)
- Branch management (branch, checkout, merge, rebase)
- Remote operations (fetch, pull, push, remote)
- History inspection (log, show, blame)
- Undoing changes (reset, revert, restore)
- Advanced operations (stash, cherry-pick, tag, reflog)
- Collaboration (bisect, submodule, worktree)

Each command includes:
- Direct jj equivalent
- Key behavioral differences
- Practical examples
- Quick reference tables

### 2. [jj-mental-model.md](./jj-mental-model.md) (21KB)
**Core concepts and architecture of jj**

**Load when:** You need to understand how jj thinks about version control, or when git-based intuition leads you astray. Essential for understanding change IDs, operation log, and the revset query language.

Coverage:
- Core architecture (CLI, Library, Backend layers)
- Change ID concept (stable change tracking)
- Working copy semantics (automatic snapshotting)
- Operation log (true undo/redo system)
- Revset language (powerful query syntax)
- First-class conflicts (conflicts as data)
- Colocated repositories (working with git)
- Object model relationships

Essential for understanding how to think in jj, not just command mappings.

### 3. [jj-setup-guide.md](./jj-setup-guide.md) (20KB)
**Installation, configuration, and setup**

**Load when:** You're installing jj for the first time, configuring editors/tools, setting up authentication, or migrating from git. Includes platform-specific installation and environment configuration.

Coverage:
- Installation (Linux, macOS, Windows)
- Initial configuration (user identity, config files)
- Git integration (colocated repos, remotes)
- Aliases and shortcuts
- Editor integration (VS Code, Vim, Emacs, etc.)
- Shell integration (completions and prompts)
- Diff/merge tools configuration
- Templates (customizing log output)
- Authentication (SSH, HTTPS, GPG signing)
- Migration strategies (gradual adoption)

Everything needed to get jj running in your environment.

### 4. [jj-powerful-features.md](./jj-powerful-features.md) (21KB)
**Unique features impossible or difficult in git**

**Load when:** You want to understand what makes jj special, or when you encounter jj features that have no git equivalent (like committing conflicts or the operation log). Read this to leverage jj's full power.

Coverage:
- Working copy as a commit
- First-class conflicts (commit and resolve later)
- Operation log and undo (reversible operations)
- Automatic rebase (descendants update)
- Change vs revision model
- Anonymous branches (optional naming)
- Move and squash operations
- Multi-workspace support
- Revset language power
- Philosophy differences

Demonstrates why jj enables workflows git cannot support.

### 5. [jujutsu-workflows.md](./jujutsu-workflows.md) (30KB)
**Practical workflows and best practices**

**Load when:** You need step-by-step guidance for specific tasks (code review, conflict resolution, history editing), or when learning best practices for daily development, collaboration, or working with large repositories.

Coverage:
- Daily development workflows
- Branching strategies (anonymous + bookmarks)
- Code review workflows (single + stacked)
- Collaboration patterns (with git users)
- Conflict resolution workflows
- History editing (safe rewriting)
- Large repository strategies
- CI/CD integration
- Common pitfalls and gotchas
- Week-by-week migration guide

Real-world usage patterns with command sequences and examples.

### 6. [jj-stacked-prs-guide.md](./jj-stacked-prs-guide.md) (NEW!)
**Comprehensive guide to bookmark-based stacked pull requests**

**Load when:** You need to break a large feature into multiple reviewable PRs with dependencies, manage a stack of changes with a linear history, or coordinate work across multiple related pull requests using bookmarks.

Coverage:
- Stack structure and naming (TASK-ID/NN-description pattern)
- Initial stack setup with root and dev bookmarks
- Creating stacked changes using `jj new -B` and `squash --into`
- Adding work to existing layers at the right position
- Handling external changes from collaborators
- Conflict management strategies
- Publishing dependent PRs on GitHub
- Stack maintenance and cleanup
- Complete example workflow from start to merge
- Advanced techniques (cherry-picking, splitting, templates)

Essential guide for managing complex features as a series of dependent, reviewable changes.

### 7. [jj-convert-to-stacked-prs.md](./jj-convert-to-stacked-prs.md) (NEW!)
**Converting existing branches into stacked PRs**

**Load when:** You have an existing branch with many commits that needs to be broken into reviewable stacked PRs, when commits don't align with desired layer structure and need reorganization, or when you need to refactor history while converting to stacks.

Coverage:
- Analysis phase (reviewing commits, identifying layers)
- Three conversion strategies (bottom-up, top-down, surgical)
- Moving changes between commits with `jj squash` and `jj restore`
- Splitting monolithic commits across layers with `jj split`
- Merging related commits into single layers
- Handling common scenarios (wrong order, bug fixes, refactoring)
- Validation and testing at each layer
- Complete examples (simple and complex conversions)
- Troubleshooting reorganization issues

Essential for transforming messy development branches into clean, reviewable stacks.

## Common Workflows

### Starting a New Project
```bash
# Initialize jj repo
jj git init myproject
cd myproject

# Or clone existing
jj git clone https://github.com/user/repo
```

→ See: [jj-setup-guide.md](./jj-setup-guide.md) § Git Integration - Load when setting up a new jj repository or configuring git interoperability.

### Daily Development
```bash
# Check status
jj status

# Make changes (automatically tracked)
# Edit files...

# Describe changes
jj describe -m "Add feature X"

# Create new change
jj new
```

→ See: [jujutsu-workflows.md](./jujutsu-workflows.md) § Daily Development Workflows - Load when learning the basic day-to-day cycle of making changes, describing them, and creating new changes.

### Code Review
```bash
# Create stacked changes
jj new -m "Part 1: Infrastructure"
# Work...
jj new -m "Part 2: Implementation"
# Work...
jj new -m "Part 3: Tests"

# Push for review
jj git push --change @
```

→ See: [jujutsu-workflows.md](./jujutsu-workflows.md) § Code Review Workflows - Load when preparing changes for code review, especially for creating and managing stacked changes.
→ See: [jj-stacked-prs-guide.md](./jj-stacked-prs-guide.md) - Load for comprehensive bookmark-based stacked PR workflow with TASK-ID organization.

### Conflict Resolution
```bash
# Conflicts are first-class - you can commit them
jj new
# Make conflicting changes

# Resolve later
jj resolve
```

→ See: [jj-powerful-features.md](./jj-powerful-features.md) § First-Class Conflicts - Load to understand how jj treats conflicts as data that can be committed and propagated.
→ See: [jujutsu-workflows.md](./jujutsu-workflows.md) § Conflict Resolution - Load for step-by-step guidance on resolving conflicts in jj.

### History Editing
```bash
# Edit any commit
jj edit REVISION

# Split commit
jj split

# Move changes between commits
jj squash --from REV1 --into REV2

# Undo mistakes
jj undo
```

→ See: [jujutsu-workflows.md](./jujutsu-workflows.md) § History Editing - Load when you need to modify, split, or reorganize commits safely using jj's operation log.

### Working with Git Users
```bash
# Colocated repo (jj + git in same dir)
jj git init --colocate

# Changes sync automatically
jj git fetch
jj git push
```

→ See: [jujutsu-workflows.md](./jujutsu-workflows.md) § Collaboration Patterns - Load when working with teams, especially when collaborating with git users or setting up colocated repositories.

## Key Concepts

### 1. No Staging Area
In jj, changes in your working copy are automatically tracked. There's no `git add` step.

→ See: [jj-mental-model.md](./jj-mental-model.md) § Working Copy Semantics - Load to understand how jj's automatic snapshotting eliminates the staging area.

### 2. Change IDs vs Commit IDs
Changes have stable IDs that persist across rewrites. Commit IDs change when you modify commits.

→ See: [jj-mental-model.md](./jj-mental-model.md) § The Change ID Concept - Load to understand how change IDs remain stable across rewrites while commit IDs change.

### 3. Operation Log
Every jj operation is recorded. You can undo/redo anything.

```bash
jj op log        # View operation history
jj undo       # Undo last operation
jj op restore @- # Restore to specific operation
```

→ See: [jj-mental-model.md](./jj-mental-model.md) § Operation Log - Load to understand jj's foundational undo/redo system that records every operation.

### 4. Revsets
Powerful query language for selecting revisions:

```bash
jj log -r 'author(alice) & ::@'        # Alice's changes leading to HEAD
jj log -r 'description(bug) ~ empty()' # Non-empty commits mentioning "bug"
jj log -r 'mine() & bookmarks()'        # My bookmarked changes
```

→ See: [jj-mental-model.md](./jj-mental-model.md) § Revset Language - Load when you need to write complex queries to select revisions or understand revset syntax.

### 5. Conflicts as Data
Conflicts can be committed, rebased, and cherry-picked. They're not errors that block progress.

→ See: [jj-powerful-features.md](./jj-powerful-features.md) § First-Class Conflicts - Load to understand how conflicts are stored as data and can be committed, rebased, and resolved later.

## Tips for Git Users

### Mental Model Shifts
1. **No staging area** - Working copy IS a commit
2. **Branches are optional** - Work without naming overhead
3. **Rewriting is safe** - Operation log has your back
4. **Conflicts are data** - Commit now, resolve later
5. **Everything has undo** - jj undo is your friend

### Command Translation Patterns
```bash
# Git workflow
git add file.txt
git commit -m "msg"
git push

# jj workflow
# Edit file.txt (auto-tracked)
jj describe -m "msg"
jj git push
```

→ See: [jj-git-command-reference.md](./jj-git-command-reference.md) for complete mappings - Load this for comprehensive git-to-jj command translations.

### Common Gotchas
1. Working copy is `@`, not `HEAD`
2. Branches are called "bookmarks" and are optional
3. `jj new` creates a child, not a sibling
4. Remote operations use `jj git fetch/push`, not `jj fetch/push`
5. Commit hashes change when you edit commits
6. **jj ≥ 0.41:** `jj git push --all` / bulk push may **skip** private/conflict
   bookmarks instead of failing — verify remotes after push ([VERSIONS.md](./VERSIONS.md))
7. **jj ≥ 0.41:** `jj file search --pattern` defaults to **regex:** not glob
   ([VERSIONS.md](./VERSIONS.md))

→ See: [jujutsu-workflows.md](./jujutsu-workflows.md) § Common Pitfalls & Gotchas - Load when you encounter confusing behavior or errors, or to learn common mistakes to avoid.
→ See: [VERSIONS.md](./VERSIONS.md) for full 0.41+ gates.

## Migration Strategies

### Gradual Adoption (Recommended)
```bash
# Add jj to existing git repo
cd my-git-repo
jj git init --colocate

# Use jj for daily work, git for pushing
jj status
jj new
jj describe
git push  # Or: jj git push
```

Both tools work on the same repository. No team buy-in required.

→ See: [jj-setup-guide.md](./jj-setup-guide.md) § Migration Strategies - Load when planning how to transition from git to jj gradually.

### Week-by-Week Learning Plan
- **Week 1:** Basic operations (status, new, describe, log)
- **Week 2:** History navigation (prev, next, edit)
- **Week 3:** Collaboration (fetch, push, bookmarks)
- **Week 4:** Advanced features (squash, split, restore)

→ See: [jujutsu-workflows.md](./jujutsu-workflows.md) § Migration Guide - Load for a structured week-by-week learning plan for adopting jj.

## Performance Tips

For large repositories:
```bash
# Sparse checkouts
jj sparse set --add src/my-component

# Multiple workspaces
jj workspace add ../feature-workspace

# Faster diffs
jj config set --user ui.diff.tool meld
```

→ See: [jujutsu-workflows.md](./jujutsu-workflows.md) § Large Repository Strategies - Load when working with large codebases and need performance optimization techniques.

## Troubleshooting

### Common Issues

**"Change is immutable" error:**
- Use `jj new` to create a new change instead of editing immutable ones

**Conflicts everywhere after rebase:**
- Conflicts propagate in jj. Resolve at root with `jj resolve`

**Lost changes:**
- Use `jj op log` and `jj op restore` to recover

**Slow performance:**
- Check `jj debug reindex` and consider sparse checkouts

→ See: [jujutsu-workflows.md](./jujutsu-workflows.md) § Common Pitfalls - Load when troubleshooting issues or recovering from mistakes.

## Further Learning

### Official Resources
- Website: https://github.com/jj-vcs/jj
- **Version-pinned docs:** [0.40.0](https://www.jj-vcs.dev/v0.40.0/) · [0.41.0](https://www.jj-vcs.dev/v0.41.0/)
- Latest docs (moves): https://docs.jj-vcs.dev/latest/
- Tutorial: https://www.jj-vcs.dev/v0.41.0/tutorial/ (or `/v0.40.0/tutorial/` for baseline)
- **CLI version gates (this pack):** [VERSIONS.md](./VERSIONS.md)

### Community
- GitHub Discussions: https://github.com/martinvonz/jj/discussions
- Discord: https://discord.gg/dkmfj3aGQN

### Blog Posts
- "Jujutsu: A Git-Compatible VCS" - Steve Klabnik
- Various user experience posts on migration and workflows

## Summary

**Start here based on your goal:**

| Goal | Read This First | When to Load |
|------|----------------|--------------|
| I know git, want jj equivalents | [jj-git-command-reference.md](./jj-git-command-reference.md) | Load when translating git commands to jj |
| I want to understand jj deeply | [jj-mental-model.md](./jj-mental-model.md) | Load to understand core concepts and architecture |
| I need to install and configure | [jj-setup-guide.md](./jj-setup-guide.md) | Load during initial setup and configuration |
| I want practical workflows | [jujutsu-workflows.md](./jujutsu-workflows.md) | Load for step-by-step task guidance |
| I want to see what makes jj special | [jj-powerful-features.md](./jj-powerful-features.md) | Load to discover unique jj capabilities |
| I need to create stacked PRs from scratch | [jj-stacked-prs-guide.md](./jj-stacked-prs-guide.md) | Load for bookmark-based stacked PR workflow |
| I need to convert existing branch to stacked PRs | [jj-convert-to-stacked-prs.md](./jj-convert-to-stacked-prs.md) | Load to reorganize commits into clean layers |
| I'm migrating a team | [jj-setup-guide.md](./jj-setup-guide.md) § Migration + [jujutsu-workflows.md](./jujutsu-workflows.md) § Collaboration | Load for team adoption strategies |
| I hit a specific problem | [jujutsu-workflows.md](./jujutsu-workflows.md) § Common Pitfalls | Load when troubleshooting errors |

**Remember:** jj is designed to be forgiving. The operation log (`jj op log`) records everything, so you can always undo with `jj undo` or restore to any previous state with `jj op restore`. Experiment fearlessly.

**Core workflow pattern:**
1. `jj status` - See what's changed
2. Make your edits (automatically tracked)
3. `jj describe` - Add a message
4. `jj new` - Start next change
5. `jj log` - See your history
6. `jj git push` - Share your work

Everything else builds on this foundation.
