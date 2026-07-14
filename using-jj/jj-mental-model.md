# Jujutsu (jj) Mental Model: Core Concepts and Architecture

This document explains how to think about version control in the Jujutsu way—not just command mappings, but the underlying mental model that makes jj different from Git and other version control systems.

## Table of Contents

1. [Philosophy and Design Goals](#philosophy-and-design-goals)
2. [Core Architecture](#core-architecture)
3. [The Change ID Concept](#the-change-id-concept)
4. [Working Copy Semantics](#working-copy-semantics)
5. [Operation Log](#operation-log)
6. [Revset Language](#revset-language)
7. [First-Class Conflicts](#first-class-conflicts)
8. [Colocated Repositories](#colocated-repositories)
9. [Object Model](#object-model)
10. [Mental Model Shifts from Git](#mental-model-shifts-from-git)

---

## Philosophy and Design Goals

Jujutsu is designed around several core principles that fundamentally differ from Git:

### 1. **Changes > Commits**
In Git, you create commits. In Jujutsu, you work with *changes* that evolve over time. A commit is just one snapshot of a change at a particular point.

### 2. **Working Copy as a Commit**
Your working directory isn't a special "staging area" or "dirty state"—it's a real commit that automatically snapshots your changes. This eliminates the need for Git's index/staging area.

### 3. **Conflicts are Data, Not Errors**
Conflicts are first-class objects that can be stored, rebased, and manipulated like any other commit. You don't need to resolve them immediately.

### 4. **Operations are Recorded**
Every action you perform on the repository is logged and can be undone. Version control for your version control.

### 5. **Backend Agnostic**
The UI and algorithms are separated from storage. Today this means Git compatibility; tomorrow it could mean other backends.

---

## Core Architecture

### The Layer Model

Jujutsu has a clean separation of concerns:

```
┌─────────────────────────────────┐
│   CLI (User Interface)          │
├─────────────────────────────────┤
│   Library (jj-lib)              │
│   - Revset language             │
│   - Operation log               │
│   - Conflict resolution         │
│   - Transaction management      │
├─────────────────────────────────┤
│   Backend (Storage)             │
│   - GitBackend (production)     │
│   - SimpleBackend (testing)     │
│   - Future: native backend      │
└─────────────────────────────────┘
```

### Key Components

**Backend**: Implements storage for commits, trees, and blobs. The Git backend uses gitoxide to store data in Git format, making it fully compatible with Git tools.

**Store**: Wraps the backend and provides caching and higher-level operations. Returns wrapped objects that carry a reference to the Store itself.

**View**: A snapshot of the repository state at a specific operation, including:
- Where bookmarks (branches) point
- Which commits are visible
- Working-copy commits for each workspace
- Tag targets

**Operation**: A snapshot of a View plus metadata (timestamp, user, description) and parent operations. Operations form a DAG just like commits do.

**ReadonlyRepo**: Represents the repo at a specific operation with an immutable View.

**MutableRepo**: A mutable version with its own copy of the View that can be modified.

**Transaction**: Has a MutableRepo and operation metadata. Committing a transaction creates a new operation in the operation log.

---

## The Change ID Concept

This is perhaps the most important conceptual shift from Git.

### What is a Change?

A **change** is a logical unit of work that evolves over time. It has:
- A **change ID**: A unique, stable identifier (typically 16 bytes, shown as 12 letters k-z)
- Multiple **commits** (revisions) representing different versions of that change

### Change ID vs Commit ID

```
Change ID: unchanging identifier for a logical change
    ↓
Commits:   qpxw2knr → zkau8vpn → mtvz7qlo
           (v1)       (v2)       (v3)
```

When you:
- Amend a commit: Same change ID, new commit ID
- Rebase a commit: Same change ID, new commit ID
- Describe a commit: Same change ID, new commit ID

### Why This Matters

1. **History Rewriting is Natural**: Since the change ID stays constant, you can freely rewrite history without losing track of what changed.

2. **Automatic Rebasing**: When you rewrite a commit, Jujutsu can automatically rebase descendants because it tracks the *change*, not the specific commit.

3. **Conflict Resolution Propagation**: When you resolve a conflict in a change, that resolution can be propagated to descendants.

4. **No "Detached HEAD"**: You're always working on a change, whether it has a bookmark or not.

### Example Workflow

```bash
# Create a change
$ jj new
# Make some edits
$ jj describe -m "Add feature X"
# Change ID: abc123...
# Commit ID: def456...

# Amend the change
$ vim foo.rs
$ jj describe -m "Add feature X (improved)"
# Change ID: abc123... (SAME!)
# Commit ID: ghi789... (different)

# Reference by either ID
$ jj show abc  # using change ID
$ jj show ghi  # using commit ID
```

---

## Working Copy Semantics

### The Working Copy IS a Commit

In Jujutsu, your working directory is always a commit. This is radically different from Git:

**Git's model:**
```
Working Directory (untracked changes)
       ↓
Index/Staging Area
       ↓
Commit History
```

**Jujutsu's model:**
```
Working Copy Commit (@)
       ↓
Parent Commits
```

### Automatic Snapshotting

Almost every `jj` command:
1. **Snapshots the working copy** before running (creates/updates the working-copy commit)
2. **Executes the command** (may create new commits, rebase, etc.)
3. **Updates the working copy** to match the new state

This means:
- No `git add` needed
- No "dirty working directory" errors
- No `git stash` needed
- Changes are never lost

### Example: What Happens When You Edit Files

```bash
$ jj status
Working copy changes:
- (empty) No description set

$ echo "hello" > file.txt

$ jj status  # Automatically snapshots!
Working copy changes:
+ M file.txt
Working copy : qwyusoqk 8f1a8efd (no description set)

# The working copy commit was automatically updated!
```

### The `@` Symbol

`@` always refers to the current working-copy commit in the current workspace. It's equivalent to Git's `HEAD`, but more powerful:

```bash
$ jj log -r @       # Current working-copy commit
$ jj log -r @-      # Parent of working copy
$ jj log -r @+      # Children of working copy (if any)
$ jj log -r ::@     # Ancestors of working copy
```

### Creating New Changes

Unlike Git, creating a new commit is explicit:

```bash
# Git: commit automatically creates new commit
$ git commit -m "message"

# Jujutsu: separate operations
$ jj describe -m "message"  # Describe current change
$ jj new                     # Create new change

# Or combined:
$ jj commit -m "message"    # Shortcut: describe + new
```

### The Power of Separation

Because describing and creating are separate:

1. **Describe before finishing**: You can write a commit message while still working:
   ```bash
   $ jj describe -m "Refactor database layer"
   # Keep editing files...
   $ jj describe -m "Refactor database layer (WIP)"
   # Still editing...
   $ jj new  # Done! Start next change
   ```

2. **Create new changes anywhere**:
   ```bash
   $ jj new -A abc  # New change after abc
   $ jj new -B xyz  # New change before xyz
   # Descendants automatically rebased!
   ```

---

## Operation Log

The operation log is Jujutsu's "version control for version control."

### What Gets Logged

Every operation that modifies the repo:
- Creating/editing commits
- Rebasing
- Pulling from remotes
- Even `jj status` (it snapshots the working copy)

### Operation Structure

Each operation contains:
- **View snapshot**: Complete state of bookmarks, tags, visible heads
- **Parent operations**: Like commits, operations form a DAG
- **Metadata**: Username, hostname, timestamp, description

```bash
$ jj op log
@  a1b2c3d4 user@host 2024-01-15 10:30:00
│  describe commit 567890ab
◉  e5f6g7h8 user@host 2024-01-15 10:25:00
│  new empty commit
◉  i9j0k1l2 user@host 2024-01-15 10:20:00
│  commit working copy
```

### The Power of Operations

**1. True Undo:**
```bash
$ jj undo              # Undo last operation
$ jj undo @-           # Undo second-to-last operation
```

**2. Selective Undo:**
```bash
$ jj op restore abc    # Restore repo to operation abc
```

**3. View History at Any Operation:**
```bash
$ jj --at-op=xyz log   # See repo state at operation xyz
$ jj --at-op=@- status # See status before last operation
```

**4. Concurrent Operations:**
If you run concurrent `jj` commands (even on different machines via network filesystem), they create divergent operations:

```
     @  operation from workspace A
    /│
   / ◉  operation from workspace B
  /  │
 ◉   │  shared parent operation
```

Jujutsu detects and resolves these automatically in most cases.

### Operation Log vs Git Reflog

Git's reflog:
- Per-reference log of where a ref pointed
- Local only
- Limited retention
- Tracks ref movements

Jujutsu's operation log:
- Complete repository state at each point
- Records *all* operations
- Enables multi-machine operation
- Foundation for undo/redo

---

## Revset Language

Revsets are Jujutsu's query language for selecting commits. They're a functional language borrowed from Mercurial but significantly enhanced.

### Core Concept

A revset is an **expression that evaluates to a set of commits**. It's compositional and type-safe.

### Symbols

```bash
@              # Working copy commit
@-             # Parent of working copy
@--            # Grandparent
root()         # Virtual root commit (all 0's)
abc123         # Commit by ID prefix
xyz789         # Change by ID prefix
main@origin    # Remote-tracking bookmark
```

### Operators

**Parent/Child:**
```bash
x-             # Parents of x
x+             # Children of x
x---           # Three generations of parents
```

**Ancestors/Descendants:**
```bash
::x            # Ancestors of x (including x)
x::            # Descendants of x (including x)
..x            # Ancestors of x, excluding root
x..            # Descendants of x, excluding x
```

**Ranges:**
```bash
x::y           # Descendants of x that are ancestors of y
               # (like git's x..y --ancestry-path)
x..y           # Ancestors of y, excluding ancestors of x
               # (like git's x..y)
```

**Set Operations:**
```bash
x | y          # Union (x or y)
x & y          # Intersection (x and y)
x ~ y          # Difference (x but not y)
~x             # Complement (everything except x)
```

### Functions

**Selection:**
```bash
all()                    # All visible commits
heads(x)                 # Commits in x with no children in x
roots(x)                 # Commits in x with no parents in x
ancestors(x, depth)      # Ancestors of x up to depth
descendants(x, depth)    # Descendants of x up to depth
```

**Bookmarks/Tags:**
```bash
bookmarks()              # All bookmark targets
bookmarks(pattern)       # Bookmarks matching pattern
remote_bookmarks()       # All remote bookmark targets
tags()                   # All tag targets
```

**Content Search:**
```bash
description(pattern)     # Commits with matching description
author(pattern)          # Commits by author
files(pattern)           # Commits modifying files
diff_contains(text)      # Commits with diffs containing text
```

**Special:**
```bash
empty()                  # Commits changing no files
merges()                 # Merge commits
conflicts()              # Commits with conflicts
mine()                   # Commits by current user
```

### Composition Power

The power comes from composition:

```bash
# Commits not yet pushed to any remote
remote_bookmarks()..@

# Commits on main since branching point
fork_point(@)..@

# My commits with TODOs since last week
mine() & description("TODO") & author_date(after:"1 week ago")

# All descendants of commit abc that touch src/
abc:: & files("src/**")

# Merge commits on current branch
@ ~ ::(@-) & merges()
```

### Default Revset

`jj log` uses a default revset that's much smarter than `git log`:

```bash
@ | ancestors(immutable_heads().., 2) | heads(immutable_heads())
```

This shows:
- Your current commit (`@`)
- Open branches (commits not yet merged to immutable heads, 2 levels deep)
- The tips of immutable branches (main, tags, etc.)

---

## First-Class Conflicts

Jujutsu's handling of conflicts is fundamentally different from other VCS systems.

### Conflicts are Data

In Git: Conflict → Error → Must resolve before continuing

In Jujutsu: Conflict → Data → Resolve when ready

### How Conflicts are Stored

Conflicts aren't stored as text markers. They're stored as a logical representation:

**A commit with conflicts contains:**
- Multiple tree objects (always odd number: 2n+1)
- First tree: base state
- Subsequent pairs: diffs to apply

Example: `Trees[A, B, C, D, E]` means:
```
Result = A + (C - B) + (E - D)
```

A simple 3-way merge (Git style) is just:
```
Trees[A, B, C] = A + (C - B)
```

### Conflict Markers in Working Copy

When you check out a conflicted commit, Jujutsu materializes conflicts using markers:

```
 <<<<<<< Conflict 1 of 1
%%%%%%% Changes from base to side #1
 apple
-grape
+grapefruit
 orange
+++++++ Contents of side #2
APPLE
GRAPE
ORANGE
 >>>>>>> Conflict 1 of 1 ends
```

This format shows:
- A snapshot (side #2)
- A diff to apply (base → side #1)
- How to resolve: apply the diff to the snapshot

### Why This Matters

**1. Conflicts Can Be Rebased:**
```bash
$ jj rebase -r conflicted_commit -d new_base
# Conflict is rebased, not re-merged
# Still just one conflict to resolve
```

**2. Partial Resolution:**
You can resolve some files and leave others conflicted.

**3. Conflict Resolution is Preserved:**
When you rebase a commit that resolved a conflict, the resolution is preserved.

**4. No "Continue" Commands:**
```bash
# Git:
$ git rebase main
# ... conflict ...
$ # edit files
$ git add .
$ git rebase --continue

# Jujutsu:
$ jj rebase -r @ -d main
# ... conflict ...
$ # edit files whenever
$ jj describe -m "resolved"
# Done! No "continue" needed
```

### Conflict Simplification

Jujutsu automatically simplifies conflict expressions:

```
Example: Rebase a conflict
  Original: C + (B - A)       # Conflict on C
  Rebased:  D + ((C + (B - A)) - C)
  Simplified: D + (B - A)     # Clean 3-way merge!
```

This prevents nested conflicts and keeps conflict resolution manageable.

### Automatic Resolution

If all sides make the same change, Jujutsu automatically resolves it. This is called the "same-change rule."

---

## Colocated Repositories

Jujutsu can work alongside Git in the same directory.

### What is Colocation?

A colocated repo has:
```
.git/          # Git's data
.jj/           # Jujutsu's data (points to .git/)
```

### Setting Up

```bash
# In existing Git repo:
$ jj git init --colocate

# Or clone with colocated repo:
$ jj git clone --colocate https://github.com/user/repo
```

### How It Works

**Automatic Sync:**
1. Every `jj` command snapshots working copy
2. Imports any changes from `.git/`
3. Executes the Jujutsu command
4. Exports visible commits to `.git/`

**Using Both Tools:**
```bash
$ jj new -m "feature: add widget"
# ... edit files ...
$ git status  # Shows changes
$ git log     # Shows jj's commits
$ jj log      # Shows same commits with change IDs
```

### What Jujutsu Adds

Even in colocated repos, Jujutsu provides:
- Change IDs (stored in `.jj/`)
- Operation log (stored in `.jj/`)
- Conflict state (exported as conflict markers to Git)
- Anonymous branches (Git sees them as commits)

### Git Compatibility

**What's Compatible:**
- All commits (use same SHAs)
- All files (same tree structure)
- Pushing/pulling to remotes
- Tags
- Working with Git tools

**What's Different:**
- Jujutsu doesn't use Git's index
- Anonymous branches don't create Git branches
- Change IDs only exist in Jujutsu
- Conflicts can be committed (exported as marker files)

---

## Object Model

Understanding the relationships between Jujutsu's objects is key to the mental model.

### The Core Objects

```
Repository
    ↓
Operation Log (DAG of Operations)
    ↓
Operation
    ├── View (snapshot at this operation)
    │   ├── Bookmarks → Commit IDs
    │   ├── Tags → Commit IDs
    │   ├── Visible Heads (anonymous branches)
    │   └── Working-copy commits per workspace
    ├── Parent Operations
    └── Metadata (user, time, description)

Commits (DAG)
    ├── Commit ID (hash)
    ├── Change ID (stable)
    ├── Tree (content)
    ├── Parents
    └── Metadata

Tree
    ├── Files → Blobs
    └── Subdirs → Trees

Workspace
    ├── Working Copy (files on disk)
    └── Pointer to Repository
```

### Visibility Model

**Visible Commits:**
- Reachable from any head in the View
- What you see in `jj log`
- Can be referenced by change ID

**Hidden Commits:**
- Old versions of rewritten commits
- Abandoned commits
- Still in repo, accessible by commit ID
- Not in `jj log` by default

### Evolution Model

```
Change abc123
    ↓
Commit v1 (commit-id: aaa)
    ↓ (amended)
Commit v2 (commit-id: bbb)  ← visible
    ↓ (rebased)
Commit v3 (commit-id: ccc)  ← visible (divergent!)
```

When multiple commits share a change ID and are visible, it's a **divergent change**.

### Workspaces

Multiple working copies can share one repo:

```
Main Workspace
    ├── .jj/ (contains full repo)
    └── files/

Other Workspace
    ├── .jj/ (pointer to main)
    └── files/
```

Each workspace has its own working-copy commit.

---

## Mental Model Shifts from Git

### 1. **Commit ≠ Snapshot**

**Git thinking:** A commit is a snapshot of your code.

**Jujutsu thinking:** A commit is one version of a change. The change is the important thing.

### 2. **Working Directory ≠ Pending Changes**

**Git thinking:** Working directory and index are "outside" the commit graph.

**Jujutsu thinking:** Working directory IS a commit in the graph.

### 3. **Branch ≠ Required**

**Git thinking:** You must be on a branch. Detached HEAD is scary.

**Jujutsu thinking:** Anonymous branches are normal. Bookmarks are optional labels.

### 4. **Conflict ≠ Error**

**Git thinking:** Conflicts must be resolved immediately.

**Jujutsu thinking:** Conflicts are just data. Resolve when convenient.

### 5. **Rebase ≠ Dangerous**

**Git thinking:** Rebase requires care. Interactive rebase is complex.

**Jujutsu thinking:** Rebasing is automatic and safe. Descendants are updated automatically.

### 6. **Undo ≠ Guess What Happened**

**Git thinking:** Use reflog, hope you find it, pray you understand the state.

**Jujutsu thinking:** Operation log shows exactly what happened. Undo is trivial.

### 7. **History ≠ Sacred**

**Git thinking:** Published history shouldn't change.

**Jujutsu thinking:** History is your tool. Rewrite it to tell the right story.

### Command Translation (with Mental Model)

| Git Command | Jujutsu Command | Mental Model Difference |
|-------------|----------------|------------------------|
| `git add` | (none) | No staging; working copy is always a commit |
| `git commit` | `jj commit` | Describes current change + creates new change |
| `git checkout` | `jj new` | Creates new change, not just switching |
| `git merge` | `jj new A B` | New change with multiple parents |
| `git rebase` | `jj rebase` | Automatic for descendants, conflict-aware |
| `git stash` | (none) | Working copy is a commit; just `jj new` |
| `git reflog` | `jj op log` | Complete operation history, not just refs |
| `git cherry-pick` | `jj rebase -r` | Rebases the change |
| `git reset --hard` | `jj restore` | Restore working copy or revisions |

---

## Conclusion

The Jujutsu mental model centers on a few key insights:

1. **Changes are the unit of work**, not commits
2. **The working copy is a commit**, not a special state
3. **Conflicts are data**, not errors
4. **Operations are recorded**, enabling true undo
5. **History is malleable**, with automatic propagation
6. **Revsets are queries**, enabling powerful selection

Understanding these concepts allows you to work with Jujutsu naturally, rather than fighting against it by trying to map Git workflows directly. The mental shift is significant, but the result is a version control system that gets out of your way and lets you focus on your code.

---

## Further Reading

- [Official Jujutsu Documentation](https://jj-vcs.dev/)
- [Jujutsu Tutorial](https://jj-vcs.dev/latest/tutorial/)
- [Glossary](https://jj-vcs.dev/latest/glossary/)
- [Git Comparison](https://jj-vcs.dev/latest/git-comparison/)
- [Steve Klabnik's Jujutsu Tutorial](https://steveklabnik.github.io/jujutsu-tutorial/)
- [Chris Krycho's "jj init"](https://v5.chriskrycho.com/essays/jj-init/)
