# Jujutsu (jj) to Git Command Reference

A comprehensive mapping of Git commands to their Jujutsu (jj) equivalents, including key differences in behavior and philosophy.

## Table of Contents

- [Repository Initialization and Cloning](#repository-initialization-and-cloning)
- [Basic Workflow](#basic-workflow)
- [Branch Management](#branch-management)
- [Remote Operations](#remote-operations)
- [History and Inspection](#history-and-inspection)
- [Undoing Changes](#undoing-changes)
- [Advanced Operations](#advanced-operations)
- [Collaboration](#collaboration)
- [Key Philosophical Differences](#key-philosophical-differences)

---

## Repository Initialization and Cloning

### git init
**jj equivalent:** `jj git init [--no-colocate]`

**Key differences:**
- By default, `jj git init` creates a "colocated" repository where jj and git share the same `.git` directory
- Use `--no-colocate` to create a separate jj-only repository that tracks a git repository
- Colocated repos allow you to use both `git` and `jj` commands interchangeably

**Examples:**
```bash
# Create a new jj repository (colocated with git)
jj git init

# Create a separate jj repository
jj git init --no-colocate
```

### git clone
**jj equivalent:** `jj git clone <source> <destination> [--remote <remote>]`

**Key differences:**
- Only supports cloning Git repositories (for now)
- Uses `--remote` instead of `--origin` to specify remote name
- Creates a colocated repository by default

**Examples:**
```bash
# Clone a repository
jj git clone https://github.com/user/repo.git

# Clone with custom remote name
jj git clone https://github.com/user/repo.git --remote upstream
```

---

## Basic Workflow

### git add
**jj equivalent:** No direct equivalent - changes are tracked automatically

**Key differences:**
- Jujutsu has **no staging area/index**
- The working copy is automatically snapshotted as a commit
- Simply modify files and they're automatically included in the working-copy commit

**Examples:**
```bash
# Add a new file (git)
touch filename
git add filename

# Add a new file (jj) - just create it!
touch filename
# That's it! The file is automatically tracked in the working copy commit
```

### git commit
**jj equivalent:** `jj commit` or `jj describe`

**Key differences:**
- `jj commit` creates a new working-copy commit on top of the current one
- `jj describe` sets/updates the description (commit message) of the current change
- The working copy is always a commit - no separate staging step needed
- Changes are automatically committed to the working copy

**Examples:**
```bash
# Finish current change and start a new one
jj commit -m "Add feature X"

# Set description of current working copy commit
jj describe -m "Work in progress on feature X"

# Interactively edit the commit message
jj describe
```

### git status
**jj equivalent:** `jj st` (short for `jj status`)

**Key differences:**
- Shows the current working-copy commit
- No concept of "staged" vs "unstaged" changes
- Shows parent commits and descendants

**Examples:**
```bash
# Show current repository status
jj st

# Shows something like:
# Working copy changes:
# M file.txt
# Parent commit: abc123 "Previous commit"
```

### git diff
**jj equivalent:** `jj diff`

**Key differences:**
- Shows diff of the working-copy commit by default (not working directory vs HEAD)
- Can easily diff any two commits without complex `^` syntax
- Use `-r` to specify a specific revision, `--from` and `--to` for ranges

**Examples:**
```bash
# Show diff of current working copy
jj diff

# Show diff of a specific commit
jj diff -r abc123

# Show diff from one commit to another
jj diff --from abc123 --to def456

# Show diff from commit to working copy
jj diff --from abc123

# Show all changes in a range (like git diff A...B)
jj diff -r A..B
```

---

## Branch Management

### Understanding Branches in Jujutsu

**Major philosophical difference:**
- Git: Branches are fundamental, HEAD is usually attached to a branch
- Jujutsu: Commits are fundamental, branches (called "bookmarks") are optional labels
- Jujutsu operates in what Git calls "detached HEAD" mode by default
- There is **no "current branch"** in jj - you work on commits directly

### git branch
**jj equivalent:** `jj bookmark list` (or `jj b l`)

**Key differences:**
- Jujutsu uses the term "bookmarks" instead of "branches"
- Bookmarks are just named pointers to commits, not required for work
- No concept of "current branch" - bookmarks are moved explicitly

**Examples:**
```bash
# List all bookmarks
jj bookmark list

# Shorter form
jj b l
```

### git checkout / git switch
**jj equivalent:** `jj new <revision>`

**Key differences:**
- `jj new` creates a **new empty commit** on top of the target
- This is fundamentally different from git checkout - you're not "checking out" a commit
- The old working-copy commit remains in history (can be restored with `jj edit`)
- Creates a new working-copy commit for your next changes

**Examples:**
```bash
# Start new work on top of main bookmark
jj new main

# Start new work on top of a specific revision
jj new abc123

# Start new work on top of multiple parents (creates a merge commit)
jj new @- main
```

### git checkout -b / git switch -c
**jj equivalent:** `jj new <revision>` + `jj bookmark create`

**Key differences:**
- Two separate operations in jj: creating a new commit and creating a bookmark
- Bookmarks are optional - you can work without them

**Examples:**
```bash
# Create new commit on top of main
jj new main

# Create a bookmark pointing to current commit
jj bookmark create feature-x

# Or create bookmark on a specific revision
jj bookmark create feature-x -r abc123
```

### git merge
**jj equivalent:** `jj new <commit1> <commit2>`

**Key differences:**
- Creates a new commit with multiple parents
- Conflicts are recorded in the commit, not left in working directory
- No special "merge" operation - just create a commit with multiple parents

**Examples:**
```bash
# Merge branch-a into current work
jj new @ branch-a

# This creates a new commit with both @ (current) and branch-a as parents
```

### git rebase
**jj equivalent:** `jj rebase`

**Key differences:**
- Descendants are **automatically rebased** when you modify any commit
- Use `-s` to rebase a source commit and its descendants
- Use `-b` to rebase a branch (a commit and its ancestors)
- Use `-r` to rebase a single commit (and its descendants)

**Examples:**
```bash
# Move change X and its descendants onto Y
jj rebase -s X -d Y

# Move bookmark A onto bookmark B
jj rebase -b A -d B

# Reorder commits: move C to before B (in sequence A-B-C-D)
jj rebase -r C --before B
```

### git branch -f / Moving branches
**jj equivalent:** `jj bookmark move`

**Key differences:**
- Bookmarks must be moved explicitly - they don't move automatically
- Use `--allow-backwards` to move bookmark to an ancestor

**Examples:**
```bash
# Move bookmark forward
jj bookmark move main --to abc123

# Shorter form
jj b m main -t abc123

# Move bookmark backward (requires flag)
jj bookmark move main --to abc123 --allow-backwards
```

### git branch -d
**jj equivalent:** `jj bookmark delete`

**Examples:**
```bash
# Delete a bookmark
jj bookmark delete feature-x

# Shorter form
jj b d feature-x
```

---

## Remote Operations

### git fetch
**jj equivalent:** `jj git fetch [--remote <remote>]`

**Key differences:**
- Only supports Git remotes currently
- Fetches all branches by default
- Remote-tracking branches are visible as bookmarks with `@<remote>` suffix

**Examples:**
```bash
# Fetch from default remote
jj git fetch

# Fetch from specific remote
jj git fetch --remote origin
```

### git pull
**jj equivalent:** `jj git fetch` + `jj rebase` (or `jj new`)

**Key differences:**
- No automatic merge/rebase - two separate operations
- More explicit control over how to integrate changes
- Fetch updates remote tracking bookmarks, then you decide how to integrate

**Examples:**
```bash
# Fetch latest changes
jj git fetch

# Rebase current work onto updated main
jj rebase -d main@origin

# Or start new work on top of updated main
jj new main@origin
```

### git push
**jj equivalent:** `jj git push`

**Key differences:**
- Can push all bookmarks or specific bookmarks
- Use `--bookmark` to push a specific bookmark
- Use `--all` to push all bookmarks
- Use `--change` to create a bookmark for a specific change and push it

**Examples:**
```bash
# Push all bookmarks
jj git push --all

# Push specific bookmark
jj git push --bookmark main

# Shorter form
jj git push -b main

# Create bookmark for a change and push it
jj git push --change @
```

### git remote
**jj equivalent:** `jj git remote`

**Examples:**
```bash
# Add a remote
jj git remote add origin https://github.com/user/repo.git

# List remotes
jj git remote list

# Remove a remote
jj git remote remove origin
```

---

## History and Inspection

### git log
**jj equivalent:** `jj log`

**Key differences:**
- Shows a graph of commits by default
- Uses revsets (powerful query language) to filter commits
- Default view shows ancestors and descendants of working copy
- Much more powerful filtering with revset language

**Examples:**
```bash
# Show compact log graph (default)
jj log

# Show ancestors of current commit (like git log)
jj log -r ::@

# Show all reachable commits (like git log --all)
jj log -r 'all()'

# Show commits not on main
jj log -r 'all() ~ ::main'

# Show commits by author
jj log -r 'author("name")'

# Show commits with specific text in diff
jj log -r 'diff_contains("foo")'
```

### git show
**jj equivalent:** `jj show <revision>`

**Key differences:**
- Shows description and diff of any commit
- Works the same way for working copy and any other commit

**Examples:**
```bash
# Show current working copy
jj show @

# Show a specific commit
jj show abc123

# Show parent of working copy
jj show @-
```

### git blame
**jj equivalent:** `jj file annotate <path>`

**Examples:**
```bash
# Annotate a file
jj file annotate src/main.rs

# Short form
jj file annotate src/main.rs
```

---

## Undoing Changes

### git restore / git checkout -- file
**jj equivalent:** `jj restore <paths>`

**Key differences:**
- Restores files in the working copy from parent commit
- Can restore from any commit with `--from`
- Can restore into any commit with `--to`

**Examples:**
```bash
# Restore files from parent
jj restore file.txt

# Restore from a specific commit
jj restore --from abc123 file.txt

# Restore into a different commit
jj restore --to abc123 file.txt
```

### git reset --hard
**jj equivalent:** `jj abandon` or `jj restore`

**Key differences:**
- `jj abandon` marks current commit as abandoned (can be undone!)
- `jj restore` makes working copy empty (match parent)
- Nothing is ever lost - operation log tracks everything

**Examples:**
```bash
# Abandon current change and start fresh
jj abandon

# Make current change empty (restore all files)
jj restore

# Abandon a specific commit
jj abandon abc123
```

### git reset --soft HEAD~
**jj equivalent:** `jj squash --from @-`

**Key differences:**
- Moves changes from parent into working copy
- More explicit about what's happening

**Examples:**
```bash
# Move changes from parent into working copy
jj squash --from @-
```

### git revert
**jj equivalent:** `jj revert -r <revision> -B @`

**Key differences:**
- Creates a commit that inverts the changes
- Use `-r` to specify which commit to revert
- Use `-d` (destination) to specify where to create the revert

**Examples:**
```bash
# Revert a commit
jj revert -r abc123 -B @

# This creates a new commit that undoes changes from abc123
```

### git reset
**jj equivalent:** Various commands depending on mode

**Key differences:**
- Git's `reset` has multiple modes that map to different jj operations
- `--soft`: `jj squash --from @-`
- `--mixed`: Not needed (no staging area)
- `--hard`: `jj abandon` or `jj restore`

---

## Advanced Operations

### git stash
**jj equivalent:** `jj new @-` (create new commit on parent)

**Key differences:**
- No special stash mechanism needed
- Old working-copy commit remains as a sibling
- Can return to it with `jj edit <commit-id>`
- Can have multiple "stashes" as parallel commits

**Examples:**
```bash
# "Stash" current work
jj new @-

# This creates a new working copy on the parent
# Old working copy remains and can be accessed

# Return to "stashed" work
jj edit <old-working-copy-id>
```

### git cherry-pick
**jj equivalent:** `jj duplicate <source> -d <destination>`

**Key differences:**
- Creates a copy of a commit on top of another
- Original commit remains unchanged

**Examples:**
```bash
# Copy commit abc123 onto current commit
jj duplicate abc123 -d @

# Copy a commit to a different location
jj duplicate abc123 -d main
```

### git commit --amend
**jj equivalent:** Not needed - changes automatically amend working copy

**Key differences:**
- Working copy is automatically amended as you make changes
- Use `jj describe` to change commit message
- Use `jj squash` to move changes into parent

**Examples:**
```bash
# Changes are automatically amended as you edit files
echo "more stuff" >> file.txt
# Already amended!

# To amend parent commit instead
jj squash
```

### git commit --fixup / git rebase --autosquash
**jj equivalent:** `jj squash --into <commit>`

**Key differences:**
- Directly moves changes into target commit
- No need for separate rebase operation
- Descendants are automatically rebased

**Examples:**
```bash
# Move working copy changes into an ancestor commit
jj squash --into abc123

# Interactively choose which changes to squash
jj squash -i --into abc123
```

### git rebase -i (split commit)
**jj equivalent:** `jj split`

**Key differences:**
- Interactively split working copy into two commits
- Can split any commit with `-r`
- Much simpler than interactive rebase

**Examples:**
```bash
# Split working copy into two commits
jj split

# Split a specific commit
jj split -r abc123
```

### git tag
**jj equivalent:** Git tags can be managed with `jj git` subcommands

**Examples:**
```bash
# Tags work through the underlying git repository
# Use git commands in colocated repos, or:
jj git export  # Export tags to git
jj git import  # Import tags from git
```

### git reflog
**jj equivalent:** `jj op log` (operation log)

**Key differences:**
- **Much more powerful than reflog**
- Tracks all operations, not just ref updates
- Records atomic updates to all refs at once
- Powers the undo/redo functionality

**Examples:**
```bash
# View operation history
jj op log

# Shows all operations performed on the repo
# Each operation has an ID that can be used to restore state
```

---

## Collaboration

### git bisect
**jj equivalent:** Not yet implemented

**Note:** Bisect functionality is planned but not yet available in jj.

### git submodule
**jj equivalent:** Not yet fully implemented

**Note:** Git submodule support is in development. See the [design docs](https://docs.jj-vcs.dev/latest/design/git-submodules/).

### git worktree
**jj equivalent:** `jj workspace` (experimental)

**Key differences:**
- Multiple workspaces can share the same repository
- Each workspace has its own working copy
- Operation log is shared across workspaces

**Examples:**
```bash
# Create a new workspace
jj workspace add ../other-workspace

# List workspaces
jj workspace list

# Remove a workspace
jj workspace forget ../other-workspace
```

---

## Key Philosophical Differences

### 1. Working Copy is a Commit

**Git:** Working directory + staging area + HEAD commit
**Jujutsu:** Working copy IS a commit that's automatically updated

This is the most fundamental difference. In jj:
- Changes to files are immediately part of the working-copy commit
- No separate `add` step needed
- No concept of "dirty working directory"
- Commands never fail due to uncommitted changes

### 2. No Staging Area / Index

**Git:** Working directory → staging area (index) → commit
**Jujutsu:** Working copy commit → new commit

The staging area doesn't exist in jj because:
- The working copy is already a commit
- You can use `jj split` to split changes into multiple commits
- You can use `jj squash -i` to selectively move changes to parent
- These provide the same workflow flexibility without the complexity

### 3. Branches are Optional (Bookmarks)

**Git:** Branches are fundamental, detached HEAD is "unusual"
**Jujutsu:** Commits are fundamental, bookmarks are optional labels

In jj:
- You work on commits directly (always "detached")
- Bookmarks are just named pointers, not required
- No "current branch" - bookmarks don't move automatically
- Jujutsu tracks all visible heads without needing branches

### 4. Conflicts are First-Class Objects

**Git:** Conflicts block operations, must be resolved immediately
**Jujutsu:** Conflicts are recorded in commits, resolved later

In jj:
- Conflicts are stored in commits and can be committed
- You can rebase conflicts and conflict resolutions
- No interrupted operations - everything succeeds
- Much better handling of merge commits ("no evil merges")

### 5. Automatic Rebasing

**Git:** Rebasing is explicit and manual
**Jujutsu:** Descendants are automatically rebased

When you modify a commit in jj:
- All descendants are automatically rebased
- Bookmarks are automatically updated
- Working copy is updated if affected
- Much more streamlined workflow for patch-based development

### 6. Operation Log for Everything

**Git:** Reflog tracks ref updates (per-ref, local only)
**Jujutsu:** Operation log tracks all operations atomically

The operation log:
- Records every operation on the repo
- Atomic updates to all refs together
- Powers undo/redo functionality
- Makes debugging much easier ("what did I just do?")

### 7. Single Root Commit

**Git:** Repositories can have multiple roots, "orphan branches"
**Jujutsu:** Single virtual root commit (all zeros hash)

Benefits:
- No "unborn branch" state
- No need for `--root` or `--orphan` flags
- Simpler mental model
- All commits have a common ancestor

### 8. Evil Merges are Welcome

**Git:** Discouraged, poorly supported historically
**Jujutsu:** Fully supported, first-class feature

In jj:
- Merge commits can contain changes not in any parent
- Changes in a commit are always relative to auto-merged parents
- `jj show` displays them correctly
- Rebasing preserves them correctly

---

## Common Workflows Comparison

### Starting a new feature

**Git:**
```bash
git checkout -b feature-x
# ... make changes ...
git add .
git commit -m "Add feature x"
```

**Jujutsu:**
```bash
jj new main  # Start new work on main
# ... make changes ...
jj commit -m "Add feature x"
# Or just: jj describe -m "Add feature x"
```

### Amending the last commit

**Git:**
```bash
# ... make more changes ...
git add .
git commit --amend
```

**Jujutsu:**
```bash
# ... make more changes ...
# Already amended! Or explicitly:
jj describe  # Update commit message
```

### Fixing an older commit

**Git:**
```bash
# ... make changes ...
git add .
git commit --fixup=abc123
git rebase --autosquash abc123^
```

**Jujutsu:**
```bash
# ... make changes ...
jj squash --into abc123
# Done! Descendants automatically rebased
```

### Splitting a commit

**Git:**
```bash
git rebase -i HEAD~3
# Mark commit as "edit"
# ... git rebase process ...
git reset HEAD^
git add -p
git commit -m "Part 1"
git add -p
git commit -m "Part 2"
git rebase --continue
```

**Jujutsu:**
```bash
jj split -r abc123
# Interactive UI to split the commit
# Done! Descendants automatically rebased
```

### Syncing with remote

**Git:**
```bash
git fetch origin
git rebase origin/main
git push origin feature-x
```

**Jujutsu:**
```bash
jj git fetch
jj rebase -d main@origin
jj git push -b feature-x
# Or: jj git push --change @ (creates bookmark automatically)
```

---

## Revset Language (jj's Superpower)

Jujutsu uses a powerful query language called "revsets" to select commits. This is much more powerful than git's revision selection.

### Common Revset Examples

```bash
# Current commit
@

# Parent of current commit
@-

# Ancestors of current commit
::@

# Descendants of a commit
abc123::

# All commits
all()

# Commits reachable from main
::main

# Commits not on main
all() ~ ::main

# Commits by author
author("Alice")

# Commits with text in message
description("bug fix")

# Commits with text in diff
diff_contains("TODO")

# Commits in a range
abc123..def456

# Merge commits
merges()

# Root commit
root()

# Visible commits (not hidden/abandoned)
visible()
```

### Using Revsets in Commands

```bash
# Log commits not on main
jj log -r 'all() ~ ::main'

# Diff all changes in a range
jj diff -r abc123::def456

# Abandon all commits by a specific author
jj abandon -r 'author("Bob")'

# Show all merge commits
jj log -r 'merges()'
```

---

## Tips for Git Users

1. **Forget about the staging area** - it doesn't exist. Just edit files.

2. **Think in commits, not branches** - work on commits directly, use bookmarks when you need to share work.

3. **Don't fear conflicts** - they're just recorded in commits, resolve them whenever convenient.

4. **Use operation log** - `jj op log` and `jj undo` are your safety net.

5. **Embrace automatic rebasing** - descendants are updated automatically when you modify commits.

6. **Learn revsets** - they're much more powerful than git's revision syntax.

7. **Use `jj squash`** - it replaces `git add -p; git commit --amend`, `git commit --fixup`, and more.

8. **Try colocated repos** - you can use both `git` and `jj` commands on the same repo while learning.

9. **Experiment freely** - everything is tracked in the operation log, you can always undo.

10. **Read the docs** - jj has excellent documentation at https://docs.jj-vcs.dev/

---

## Further Reading

- [Official Jujutsu Documentation](https://docs.jj-vcs.dev/)
- [Git Comparison](https://docs.jj-vcs.dev/latest/git-comparison/)
- [Tutorial](https://docs.jj-vcs.dev/latest/tutorial/)
- [Revsets Reference](https://docs.jj-vcs.dev/latest/revsets/)
- [Steve's Jujutsu Tutorial](https://steveklabnik.github.io/jujutsu-tutorial/)
- [Chris Krycho's "jj init"](https://v5.chriskrycho.com/essays/jj-init/)

---

## Quick Reference Card

| Git Command | Jujutsu Command | Notes |
|------------|----------------|-------|
| `git init` | `jj git init` | Colocated by default |
| `git clone` | `jj git clone` | |
| `git add` | N/A | Automatic |
| `git commit` | `jj commit` | Finishes current change |
| `git commit --amend` | N/A | Automatic |
| `git status` | `jj st` | |
| `git diff` | `jj diff` | |
| `git log` | `jj log` | |
| `git show` | `jj show` | |
| `git branch` | `jj bookmark list` | |
| `git checkout` | `jj new` | Creates new commit |
| `git merge` | `jj new A B` | Multi-parent commit |
| `git rebase` | `jj rebase` | Auto-rebases descendants |
| `git fetch` | `jj git fetch` | |
| `git pull` | `jj git fetch` + `jj rebase` | Two steps |
| `git push` | `jj git push` | |
| `git stash` | `jj new @-` | Old commit remains |
| `git cherry-pick` | `jj duplicate` | |
| `git revert` | `jj revert` | |
| `git reset --hard` | `jj abandon` or `jj restore` | |
| `git reflog` | `jj op log` | Much more powerful |
| `git bisect` | Not yet | Planned feature |
| `git blame` | `jj file annotate` | |

---

*This reference is based on Jujutsu 0.24+ and may evolve as the tool develops.*
