# Powerful Features in Jujutsu (jj) vs Git

This document explores powerful features in Jujutsu that are either impossible or extremely difficult to achieve in Git, organized by feature category with concrete examples and use cases.

## 1. Working Copy Model: Working Copy as a Commit

### The Core Difference

**Jujutsu**: The working copy IS a commit. Every change is automatically snapshotted and recorded as a new version of the working copy commit.

**Git**: The working copy is separate from commits, with a staging area (index) that acts as a buffer.

### Why This Is Powerful

1. **No Lost Work**: Since every `jj` command snapshots your working copy first, you can never lose uncommitted work. No need for `git stash`.

2. **Describe Before Complete**: You can write a commit message for work-in-progress and keep working:
   ```bash
   # Write commit message now, keep working
   jj describe -m "Add authentication feature"
   # Make more changes...
   # Still the same commit, just evolved
   ```

3. **No Staging Confusion**: There's no "modified, staged, unstaged" mental model. Just: "what's in my working directory is the current commit."

4. **Automatic History**: Every snapshot is preserved, viewable via `jj obslog` (obsolescence log):
   ```bash
   jj obslog  # See how the current change evolved over time
   ```

### Impossible in Git

Git fundamentally separates:
- Unstaged changes (working directory)
- Staged changes (index)
- Committed changes (history)

This three-way split creates friction and makes recovering intermediate states nearly impossible.

## 2. First-Class Conflicts

### The Core Difference

**Jujutsu**: Conflicts are stored as semantic objects in commits, with full understanding of all conflict sides.

**Git**: Conflicts are text markers that must be resolved before continuing.

### Why This Is Powerful

1. **Rebase Through Conflicts**: You can rebase a commit with conflicts, and the conflict resolution propagates:
   ```bash
   # Commit A has a conflict
   jj rebase -r A -d B
   # The conflict is rebased too, you can resolve it later
   ```

2. **Automatic Conflict Resolution Propagation**: Resolve a conflict once, and when you rebase descendants, the resolution is automatically applied:
   ```bash
   # Initial conflict in commit X
   jj new X  # Create child to resolve
   # Resolve conflict
   jj squash  # Move resolution into X

   # Now rebase descendants - resolution propagates automatically!
   jj rebase -r Y -d X  # Y gets the resolution too
   ```

3. **Commit Conflicted States**: You can commit a merge with conflicts and resolve it in a later commit:
   ```bash
   jj new A B  # Create merge commit
   # Conflicts appear, but operation succeeds
   jj log  # See the conflicted merge in history

   # Resolve in next commit
   jj new @
   # Resolve conflicts
   jj squash  # Move resolution to parent
   ```

4. **Multi-Way Conflicts**: Handle 3+ way merges naturally:
   ```bash
   jj new A B C D  # Merge four branches
   # Jujutsu handles arbitrary n-way merges
   ```

5. **Interactive Conflict Resolution Workflow**: Can defer, inspect, and resolve conflicts at your own pace without blocking other work.

### Impossible/Very Difficult in Git

```bash
# Git workflow - painful
git rebase feature
# Conflict! Must resolve NOW or abort
# Can't continue other work easily
# Can't commit the conflicted state
# Must resolve before each rebased commit
git rebase --continue
# Another conflict...
git rebase --continue
# ...painful iteration...

# With git rerere, you get some help, but:
# - Only for recorded conflicts
# - Resolution is hidden magic
# - No explicit conflict objects
```

## 3. Operation Log and Undo

### The Core Difference

**Jujutsu**: Every operation (commit, rebase, merge, checkout, etc.) is recorded in an operation log.

**Git**: Only has reflog, which tracks ref updates and is limited to specific branches.

### Why This Is Powerful

1. **Undo ANY Operation**: Not just commits, but rebases, merges, branch operations:
   ```bash
   jj rebase -s A -d B  # Oops, wrong target
   jj undo  # Reverts the entire rebase operation
   ```

2. **Selective Undo**: Undo a specific operation in the past without affecting later ones:
   ```bash
   jj op log  # Find the problematic operation
   jj op restore <operation>  # Jump to that state
   # Or
   jj op revert <operation>  # Undo just that operation
   ```

3. **See ALL Changes to Repo**: View every modification:
   ```bash
   jj op log
   # Shows:
   # - All commits
   # - All rebases
   # - All branch updates
   # - All merges
   # - Working copy snapshots
   # - Fetch/push operations
   ```

4. **Time Travel Debugging**: Load the repo at any previous operation:
   ```bash
   jj --at-op=<operation-id> log
   jj --at-op=<operation-id> diff
   # See the repo exactly as it was at that point
   ```

5. **Helping Others**: When a colleague says "I don't know what I did," you can:
   ```bash
   jj op log  # See their entire history
   jj op restore <before-problem>  # Fix it
   ```

### Impossible/Very Difficult in Git

```bash
# Git reflog only tracks ref updates
git reflog  # Only shows commits and checkouts
git reflog --all  # Shows all branches, but:
# - Doesn't show rebases as operations
# - Doesn't show merges as operations
# - Doesn't show working copy states
# - Per-branch, not repository-wide
# - Can't "undo a rebase" as a single operation

# To undo a rebase in Git:
git reflog branch-name
# Find SHA before rebase
git reset --hard SHA
# But you've lost any other work in between
```

## 4. Automatic Rebase and Evolution

### The Core Difference

**Jujutsu**: When you modify a commit, ALL descendants are automatically rebased.

**Git**: You must manually rebase descendants or use `git rebase --update-refs` (recent addition, still manual).

### Why This Is Powerful

1. **Insert Commits Anywhere**: Add a commit in the middle of a stack:
   ```bash
   jj new --after A --before B
   # Creates new commit between A and B
   # B and all its descendants are automatically rebased
   ```

2. **Edit Any Commit**: Modify any commit in history:
   ```bash
   jj describe -r A -m "New message"
   # All descendants of A are automatically updated

   jj new A  # Edit commit A
   # Make changes
   jj squash  # Squash into A
   # All descendants of A automatically rebased
   ```

3. **Reorder Commits Easily**:
   ```bash
   # Swap order of commits B and C where C is a child of B
   jj rebase -r C -d B-  # Move C to grandparent
   jj rebase -r B -d C   # Move B after C
   # All descendants automatically handled
   ```

4. **No Continue/Abort Dance**: No `--continue` or `--abort` needed. If there's a conflict, it's recorded and you continue.

5. **Patch-Based Workflow**: Makes maintaining patch stacks trivial:
   ```bash
   # Update base patch
   jj new base-patch
   # Edit
   jj squash
   # All dependent patches automatically rebased!
   ```

### Very Difficult in Git

```bash
# Git requires manual intervention
git rebase -i main
# Edit commit A in middle of stack
# Manually reorder and mark for edit
# git rebase --continue after each conflict
# Descendants don't auto-update

# To insert a commit in middle:
git rebase -i main
# Mark commit B for edit
git commit --amend  # Make your change
git commit  # Add new commit
git rebase --continue
# Lots of manual steps, easy to mess up

# With modern Git:
git rebase -i main --update-refs
# Better, but still manual, still interactive editor
```

## 5. Change vs Revision Model

### The Core Difference

**Jujutsu**: Distinguishes between a "change" (semantic identity) and "revisions" (specific versions).

**Git**: Only has commits (which conflate identity and content).

### Why This Is Powerful

1. **Change Identity Survives Rebases**:
   ```bash
   # Change ID: "qpvuntsm"
   jj new -m "Add feature"
   # Change ID remains "qpvuntsm"

   jj rebase -r qpvuntsm -d main
   # Still change ID "qpvuntsm", but different revision
   # Can track the semantic change across rewrites
   ```

2. **Evolution Log**: See how a change evolved:
   ```bash
   jj obslog qpvuntsm
   # Shows all revisions of this change:
   # - Initial version
   # - After edits
   # - After rebases
   # - After squashes
   ```

3. **Easier Collaboration on Changes**:
   ```bash
   # Person A creates change X
   # Person B rebases it, adds to it
   # Both can reference the same change ID
   # Even though it's been rewritten multiple times
   ```

4. **Recover Accidental Changes**: If you squashed changes into the wrong commit:
   ```bash
   jj obslog @  # See previous states
   jj new --after obslog@-  # Create from previous state
   jj squash --from @ --into correct-commit
   ```

### Impossible in Git

Git commit SHAs change with ANY modification:
- Rebase? New SHA
- Amend? New SHA
- Change message? New SHA

No way to track "this is the same logical change" across rewrites. Gerrit and Phabricator try to add this with Change-Id footers in commit messages, but it's a hack.

## 6. Anonymous Branches (Default Mode)

### The Core Difference

**Jujutsu**: Branches are optional pointers. Work happens on anonymous commits by default.

**Git**: Everything happens on named branches. Detached HEAD is treated as an error state.

### Why This Is Powerful

1. **No Branch Naming Overhead**: Start working immediately:
   ```bash
   jj new main  # Start new work, no branch name needed
   jj new  # Create another change on top
   jj new  # And another
   # No branch names, but full history
   ```

2. **Experiment Freely**: No risk of pushing wrong branch:
   ```bash
   # In Git, might accidentally push experimental branch
   git push origin feature  # Oops, pushed experiment

   # In jj, branches don't auto-follow:
   jj new feature
   # Make experimental changes
   # Can't accidentally push - branch didn't move
   ```

3. **Branches Are Just Markers**: Add names when needed:
   ```bash
   # Do work
   jj new main
   # Make changes across multiple commits

   # Name it when ready to share
   jj bookmark create feature-x
   ```

4. **Log Shows All Work**: Default log shows all recent local work:
   ```bash
   jj log
   # Shows:
   # - Your current change
   # - All local branches (named and anonymous)
   # - Recent ancestors
   # Clean, informative view
   ```

5. **Multiple Working Branches**: Easy to maintain multiple lines of work:
   ```bash
   jj new main -m "Feature A"
   # Work on A
   jj new main -m "Feature B"
   # Work on B
   # No branch names needed until ready to share
   ```

### Difficult in Git

```bash
# Git forces branch naming
git checkout -b feature-a  # Must name it
# Oops, wrong name
git branch -m feature-a better-feature-name

# Detached HEAD is scary
git checkout ABC123
# You are in 'detached HEAD' state...
# WARNING: you may lose work!

# Must constantly manage branches
git branch  # List branches
git branch -d old-feature  # Clean up
# Branch management is overhead
```

## 7. Squash (path-level moves between commits)

### The Core Difference

**Jujutsu**: Fine-grained `jj squash --from/--into` (and `jj restore`) to move changes between any commits. There is no `jj move` subcommand on 0.40+.

**Git**: Limited to staging/unstaging in working directory, or interactive rebase for history.

### Why This Is Powerful

1. **Move Changes Between Non-Adjacent Commits**:
   ```bash
   # Squash/move specific changes from commit A into commit B
   # (A and B can be anywhere in history)
   jj squash --from A --into B

   # Interactively choose hunks (-i / --interactive)
   jj squash --from A --into B -i

   # Path-limited squash into another commit
   jj squash --from A --into B path/to/file
   ```

2. **Squash with Precision**:
   ```bash
   # Squash current changes to parent
   jj squash

   # Squash interactively (select what to move)
   jj squash -i

   # Squash into arbitrary commit
   jj squash --into some-commit

   # Squash from arbitrary commit to its parent
   jj squash --from some-commit
   ```

3. **Split Better**: Split commits at any point:
   ```bash
   # Split working copy commit
   jj split

   # Split historical commit
   jj split -r some-old-commit
   # Descendants automatically rebased!
   ```

4. **Reorganize History Freely**:
   ```bash
   # Realized feature-A code got mixed into feature-B commit
   jj squash --from feature-B --into feature-A src/feature-a/
   # Done! No rebase, no conflict markers
   ```

### Very Difficult in Git

```bash
# Git can only work with index (working directory)
git add -p  # Interactive staging
# But can only move between working dir and staged

# To move changes between commits:
git rebase -i main
# Mark commits for edit
# Manually cherry-pick hunks
# git rebase --continue
# Easy to mess up, many steps

# To split a commit:
git rebase -i main
# Mark commit for edit
git reset HEAD^
git add -p  # Stage subset
git commit
git add .
git commit
git rebase --continue
# Many manual steps, error-prone
```

## 8. Multi-Workspace Support

### The Core Difference

**Jujutsu**: First-class multiple working copies (workspaces) backed by single repo.

**Git**: Has worktrees, but they're second-class and limited.

### Why This Is Powerful

1. **Easy Creation**:
   ```bash
   jj workspace add ../feature-b
   # New working copy, shares all history
   # Can have different commits checked out
   ```

2. **Independent Working Copies**: Each workspace can:
   - Check out different commits
   - Have different uncommitted changes
   - Work on different features simultaneously

3. **Stale Detection**: Jujutsu knows when a workspace is stale:
   ```bash
   # From workspace A, modify workspace B's commit
   # Workspace B becomes "stale"
   cd ../workspace-b
   jj status  # "Working copy is stale"
   jj workspace update-stale  # Updates to new state
   ```

4. **Better Than Git Worktrees**:
   - No partial clone confusion
   - Better handling of updates
   - Cleaner mental model

### Limited in Git

```bash
# Git worktrees work but have issues
git worktree add ../feature-b
# Problems:
# - Harder to manage
# - Branch restrictions
# - Easier to get confused
# - Less robust stale detection

# Can't easily tell which worktree modified what
# No "stale" concept
# Must manually clean up
```

## 9. Revset Language

### The Core Difference

**Jujutsu**: Powerful functional language for selecting sets of revisions.

**Git**: Limited revision selection syntax.

### Why This Is Powerful

1. **Compositional Selection**:
   ```bash
   # All local changes not in main
   jj log -r 'all() ~ ancestors(main)'

   # Ancestors of branches, excluding tags
   jj log -r 'ancestors(bookmarks()) ~ ancestors(tags())'

   # Working copy and its 2 ancestors
   jj log -r '@ | ancestors(@, 2)'
   ```

2. **Set Operations**:
   - Union: `A | B`
   - Intersection: `A & B`
   - Difference: `A ~ B`

3. **Functions**: Built-in functions for queries:
   - `ancestors(x)` or `::x`
   - `descendants(x)` or `x::`
   - `bookmarks()`, `tags()`, `main()`
   - `author(pattern)`, `description(pattern)`
   - `empty()`, `merges()`

4. **Range Operators**:
   ```bash
   # Ancestors of A
   ::A

   # Descendants of A
   A::

   # Between A and B (DAG range)
   A::B

   # Difference range (like git A..B)
   A..B
   ```

5. **Practical Examples**:
   ```bash
   # All my commits not in main
   jj log -r 'author(me) ~ ancestors(main@origin)'

   # Empty commits (need to add content)
   jj log -r 'empty()'

   # All merge commits
   jj log -r 'merges()'

   # Exclude gh-pages branch from view
   jj log -r 'all() ~ ancestors(gh-pages)'
   ```

### Limited in Git

```bash
# Git has basic range syntax
git log main..feature  # Commits in feature not in main
git log --all  # All refs
git log --branches  # All branches

# But:
# - No set operations (union, intersection, difference)
# - No compositional queries
# - No clean way to exclude branches from view
# - Much more limited selection capabilities

# Git's answer is often "pipe to grep"
git log --all --oneline | grep -v gh-pages
# Textual processing, not semantic queries
```

## 10. Philosophy and Design Principles

### Core Design Differences

**Jujutsu Philosophy**:
1. **Operations should be composable**: Combine operations naturally
2. **No special modes**: No "detached HEAD", no "merge in progress"
3. **Conflicts are data**: First-class objects, not error states
4. **Automatic correctness**: System maintains invariants
5. **Explicit is better**: Branch updates are explicit, not implicit

**Git Philosophy**:
1. **Commits are sacred**: Once committed, they're history
2. **Working directory is special**: Separate from committed history
3. **Conflicts are errors**: Must be resolved to continue
4. **Manual control**: User must manage many details
5. **Implicit convenience**: Many automatic behaviors

### Workflows Enabled by Jujutsu

1. **Continuous Rebase Workflow**: Keep all work rebased on trunk:
   ```bash
   # Everything auto-rebases on trunk
   jj new main
   # Work work work across many commits
   # Regularly:
   jj git fetch
   jj rebase -d main
   # All descendants automatically rebased
   # Conflicts handled gracefully
   ```

2. **Patch Stack Workflow**: Maintain dependent patch series:
   ```bash
   # Base patch
   jj new main -m "Patch 1"
   jj new -m "Patch 2"
   jj new -m "Patch 3"

   # Update patch 1
   jj new patch-1
   # Edit
   jj squash
   # Patches 2 and 3 automatically rebased!
   ```

3. **Fearless Experimentation**: Try things without branch overhead:
   ```bash
   jj new main
   # Experiment experiment experiment
   # Multiple commits, no names needed
   # If it works, name it:
   jj bookmark create feature-x
   # If not, abandon:
   jj abandon
   ```

### Impossible/Difficult in Git

These workflows exist in Git but require:
- Extensive use of interactive rebase
- Careful branch management
- Risk of losing work
- Manual conflict resolution at each step
- Using git rerere for repeated conflicts
- Complex scripts and aliases

## Performance and Safety

### Concurrent Operation Safety

**Jujutsu**: Lock-free concurrency
```bash
# Terminal 1
jj describe -m "Feature A"

# Terminal 2 (at same time)
jj new main

# Both succeed! Conflicts detected and recorded
# No corruption, no lost work
```

**Git**: Requires locks
```bash
# Terminal 1
git commit -m "Feature A"

# Terminal 2 (at same time)
git checkout -b feature

# One might fail with lock errors
# Risk of corruption with distributed filesystems
```

### Data Safety

1. **Nothing is Ever Lost**: Operation log keeps everything:
   ```bash
   # Accidentally abandoned commits
   jj op log  # Find them
   jj op restore <operation>  # Get them back
   ```

2. **Automatic Snapshots**: Working copy snapshotted constantly:
   ```bash
   jj obslog  # See all working copy states
   # Can recover any previous state
   ```

3. **No Forced Actions**: System won't let you lose data:
   ```bash
   # Git:
   git checkout branch  # Error: you have unstaged changes
   git checkout -f branch  # Force, lose changes

   # Jujutsu:
   jj new branch  # Just works, preserves changes
   # Previous working copy state preserved in obslog
   ```

## Conclusion: Why These Features Matter

The features in Jujutsu that are impossible or very difficult in Git all stem from fundamental design choices:

1. **Separate implementation from storage**: Can evolve UX without storage constraints
2. **Conflicts as data**: Enables automatic rebasing and evolution
3. **Operation log**: Makes undo/redo universal and safe
4. **Working copy as commit**: Eliminates staging confusion
5. **Revset language**: Makes complex queries simple and composable
6. **Automatic rebasing**: Maintains patch stacks with minimal effort

These aren't just "nice to have" features—they enable fundamentally different workflows that are either impossible or extremely error-prone in Git. For developers working with:
- **Patch stacks** (like kernel development)
- **Continuous rebase workflows**
- **Complex history management**
- **Collaborative conflict resolution**
- **Experimentation without branch overhead**

...Jujutsu provides capabilities that simply don't exist in Git without extensive manual work, custom scripts, or accepting significant risk of data loss or repository corruption.

The Git compatibility layer means you can use these features today while still collaborating with Git users, making adoption practical without requiring team-wide migration.
