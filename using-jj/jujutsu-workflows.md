# Jujutsu (jj) Practical Workflows Guide

A comprehensive guide to real-world workflows and best practices for Jujutsu, based on official documentation and community experience.

## Table of Contents

1. [Daily Development Workflows](#daily-development-workflows)
2. [Branching Strategies](#branching-strategies)
3. [Code Review Workflows](#code-review-workflows)
4. [Collaboration Patterns](#collaboration-patterns)
5. [Conflict Resolution](#conflict-resolution)
6. [History Editing](#history-editing)
7. [Large Repository Strategies](#large-repository-strategies)
8. [CI/CD Integration](#cicd-integration)
9. [Common Pitfalls & Gotchas](#common-pitfalls--gotchas)
10. [Migration Guide](#migration-guide)

---

## Daily Development Workflows

### Basic Daily Cycle

The fundamental jj workflow differs from Git by treating the working copy as a commit:

```bash
# Start your day - see what's new
jj log
jj status

# Fetch latest changes (no merge needed!)
jj git fetch

# Start working on a feature
jj describe -m "Add user authentication"

# Make changes to files...
# Jujutsu automatically snapshots changes on every command

# Check your progress
jj diff
jj status

# Create a new change to continue work
jj new -m "Add password validation"

# Continue making changes...
```

**Key Insight**: Unlike Git, you don't need to explicitly "commit". Every `jj` command snapshots your working copy. Use `jj new` when you want to start a new logical change.

### Working on Multiple Features

```bash
# Start feature A
jj new main -m "Feature A: Add login"
# Work on feature A...

# Need to switch to feature B?
jj new main -m "Feature B: Update homepage"
# Work on feature B...

# Back to feature A
jj edit <change-id-of-feature-A>
# Continue work on feature A...

# Or create a child of feature A
jj new <change-id-of-feature-A> -m "Feature A: Add logout"
```

### Interactive Development Pattern

Jj supports a highly interactive workflow:

```bash
# Describe your intent first (optional but helpful)
jj new -m "Refactor database layer"

# Make changes incrementally...
jj diff  # Review changes so far

# Want to split your work?
jj split -i  # Interactively choose what stays in this commit

# Need to move some changes to parent?
jj squash -i  # Interactively squash parts into parent

# Made changes in wrong commit?
jj edit <parent-commit>  # Edit directly
# or
jj new <parent-commit>   # Create new change, then squash
```

### Amending vs Creating New Changes

```bash
# Traditional Git-style: amend current work
jj squash  # Squash working copy into parent

# Jj-native style: explicit new changes
jj new -m "Next change"  # Keep changes separate

# Choosing between them:
# - Use `jj new` when changes are logically distinct
# - Use `jj squash` when refining/fixing previous change
```

### Daily Sync Pattern

```bash
# Morning sync
jj git fetch --all-remotes
jj log  # See what changed

# During the day - keep working
# Changes are automatically tracked

# End of day - push your work
jj bookmark set my-feature  # Create/update bookmark
jj git push --bookmark my-feature

# Or use the convenience flag
jj git push --change @  # Auto-creates bookmark from change ID
```

---

## Branching Strategies

### Anonymous Branches (Default Jj Style)

Jj's natural mode is **anonymous branches** - no bookmark needed until sharing:

```bash
# Create a stack of changes (no bookmarks needed)
jj new main -m "Foundation work"
# Make changes...

jj new -m "Build on foundation"
# Make changes...

jj new -m "Final polish"
# Make changes...

jj log  # Beautiful linear stack, no bookmark noise

# Only create bookmark when pushing
jj bookmark create feature-x
jj git push --bookmark feature-x
```

**When to use**: Personal development, experimentation, rapid prototyping.

### Named Bookmarks (Git Compatibility)

For teams using Git forges or requiring named branches:

```bash
# Create and track a bookmark
jj bookmark create feature/user-auth
jj bookmark set feature/user-auth  # Update to current commit

# Track remote bookmarks
jj bookmark track feature/user-auth@origin

# Update bookmark as you work
jj new -m "Add auth logic"
# Work...
jj bookmark set feature/user-auth  # Move bookmark forward

# Push
jj git push --bookmark feature/user-auth
```

**When to use**: Team collaboration, PR workflows, CI/CD integration.

### Stacked Changes Pattern

Jj excels at creating and managing stacked changes:

```bash
# Create a stack
jj new main -m "Step 1: Add database schema"
# Implement step 1...

jj new -m "Step 2: Add API endpoints"
# Implement step 2...

jj new -m "Step 3: Add UI components"
# Implement step 3...

# Visualize the stack
jj log

# Need to fix Step 1?
jj edit <step-1-change-id>
# Make fixes...

jj new  # Create new empty change as working copy

# Stack automatically rebases!
# Step 2 and Step 3 are updated with Step 1's changes
```

**Advantages**:
- Each logical change is separate
- Easy to review individual steps
- Changes can be landed incrementally
- Conflicts resolved automatically when possible

### Branch/Bookmark Management Commands

```bash
# List all bookmarks
jj bookmark list

# Create bookmark at specific revision
jj bookmark create name -r <revision>

# Move bookmark
jj bookmark set name -r <revision>

# Delete bookmark (local and remote)
jj bookmark delete name

# Forget local bookmark operations (reset to remote state)
jj bookmark forget name

# Track remote bookmark
jj bookmark track name@origin

# Untrack remote bookmark
jj bookmark untrack name@origin
```

---

## Code Review Workflows

### Preparing Changes for Review

#### Single-Commit Review

```bash
# Polish your change
jj describe -m "feat: Add user authentication

Implements OAuth2 authentication with support for:
- Google Sign-In
- GitHub OAuth
- Email/password fallback

Closes #123"

# Clean up with interactive squash if needed
jj squash -i  # Select what to keep

# Create bookmark and push
jj bookmark create auth-feature
jj git push --bookmark auth-feature

# Create PR using GitHub CLI or web UI
gh pr create --base main --head auth-feature
```

#### Stacked Review

```bash
# Create stack with meaningful descriptions
jj new main -m "refactor: Extract auth interface"
# Work...

jj new -m "feat: Add OAuth2 provider"
# Work...

jj new -m "feat: Add Google Sign-In"
# Work...

# Create bookmarks for each level
jj bookmark create auth-interface -r <step-1>
jj bookmark create oauth-provider -r <step-2>
jj bookmark create google-signin -r <step-3>

# Push all
jj git push --all

# Create PRs in order
gh pr create --base main --head auth-interface
gh pr create --base auth-interface --head oauth-provider
gh pr create --base oauth-provider --head google-signin
```

### Responding to Review Feedback

#### Amending Based on Feedback

```bash
# Reviewer asks for changes to commit X
jj new <commit-x> -m "Address review feedback"
# Make requested changes...

# Squash fixes into original commit
jj squash --into <commit-x>

# If in a stack, children auto-rebase!

# Force push (safe because jj tracks history)
jj git push --bookmark feature --force
```

#### Preserving Review Comments with Separate Commits

Some teams prefer visible "fixup" commits during review:

```bash
# Create fixup commit
jj new <reviewed-commit> -m "fixup: Address @reviewer's comments"
# Make changes...

# Push the fixup
jj bookmark set my-feature
jj git push --bookmark my-feature --force

# After approval, squash before merging
jj squash -r <fixup-commit>
jj git push --bookmark my-feature --force
```

### Managing Multiple PRs

```bash
# Work on PR 1
jj edit <pr1-commit>
# Make changes...

# Switch to PR 2
jj edit <pr2-commit>
# Make changes...

# Create new PR while others are in review
jj new main -m "New feature"
# Work...

# All PRs are independent, tracked by bookmarks
jj bookmark list
```

### Rebasing PRs on Updated Main

```bash
# Main branch moved forward
jj git fetch

# Rebase your change
jj rebase -r <your-change> -d main@origin

# If you have a stack
jj rebase -s <bottom-of-stack> -d main@origin

# Push updated version
jj git push --bookmark feature --force
```

---

## Collaboration Patterns

### Working with Git Users

Jj is fully compatible with Git - your team doesn't need to know you use jj:

```bash
# Clone team's Git repo
jj git clone git@github.com:team/repo.git

# Work with jj commands
jj new -m "My feature"
# Work...

# Push looks like normal Git
jj bookmark create my-feature
jj git push --bookmark my-feature

# Teammates see a normal Git branch and PR
```

### Colocated Repositories

Mix `jj` and `git` commands in same repo:

```bash
# Initialize colocated repo
jj git init --colocate

# Use jj for most work
jj new -m "My changes"

# Some tools require git commands
git status  # Works fine
git log     # Shows Git view

# Back to jj
jj log     # Shows jj view

# Jj automatically syncs with Git on each command
```

**Caution**: Interleaving can cause:
- Divergent changes (fixable with `jj bookmark list` and `jj bookmark set`)
- Branch conflicts
- Confusion about HEAD state

**Best Practice**: Primarily use `jj`, use `git` only for read operations or tool compatibility.

### Sharing Work in Progress

```bash
# Push WIP changes to personal branch
jj bookmark create wip/my-experiment
jj git push --bookmark wip/my-experiment

# Teammate can check it out
# In their repo:
jj git fetch
jj new wip/my-experiment@origin
```

### Collaborative Conflict Resolution

Because jj has first-class conflicts, you can:

```bash
# Person A creates conflicted merge
jj new feature-a feature-b -m "Merge A and B"
# Conflicts appear

# Push the conflicted state (!)
jj bookmark create merge-ab
jj git push --bookmark merge-ab

# Person B can help resolve
jj git fetch
jj new merge-ab@origin -m "Resolve conflicts in file X"
# Resolve conflicts...
jj squash

jj git push --bookmark merge-ab --force
```

**Note**: Only works when both users use jj. Git users will see conflict markers.

### Code Review Without PRs (Gerrit-Style)

```bash
# Each change gets its own review
jj new main -m "Change 1"
# Work...

jj new -m "Change 2"  # Based on Change 1
# Work...

# Push each change for review (using Gerrit, Phabricator, etc.)
jj git push --change <change-1>
jj git push --change <change-2>

# Update after feedback
jj edit <change-1>
# Make changes...
jj git push --change <change-1>

# Change 2 auto-rebases on Change 1 updates!
```

---

## Conflict Resolution

### Understanding Jj's Conflict System

Jj stores conflicts as first-class objects, not just text markers:

```
# Conflict visualization in jj log
×  conflicted-commit
│  (conflict) My change description
```

**Key Benefits**:
1. Conflicts can be committed and rebased
2. No "rebase hell" - conflicts can be deferred
3. Automatic conflict resolution in many cases
4. History of conflict resolution is tracked

### Basic Conflict Resolution

```bash
# Create a conflict
jj rebase -r feature-branch -d main
# Conflict appears

# Check conflict status
jj status
# Shows: "Warning: There are unresolved conflicts..."

# View conflicts in files
cat conflicted-file.txt
# Shows conflict markers

# Method 1: Resolve directly in working copy
# Edit the file, remove markers, keep desired content

jj diff  # Verify resolution
jj status  # Should show "Existing conflicts were resolved"

# Method 2: Use merge tool
jj resolve --tool meld  # or your preferred tool

# Check resolution worked
jj log  # Conflict marker (×) should be gone
```

### Conflict Markers Explained

Jj uses unique conflict markers showing diffs vs snapshots:

```text
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

**How to resolve**:
- Apply the diff (side #1 changes) to the snapshot (side #2)
- In this case: change "GRAPE" to "GRAPEFRUIT" in the uppercased version
- Result: `APPLE\nGRAPEFRUIT\nORANGE`

### Deferred Conflict Resolution

One of jj's superpowers:

```bash
# Rebase creates conflict
jj rebase -r feature -d main
# Conflict in feature

# Keep working! Create child changes
jj new -m "Build on feature (with conflicts)"
# Make changes...

jj new -m "Another change"
# Make changes...

# Stack builds on conflicted commit!
# Resolve when ready
jj edit <conflicted-commit>
# Resolve conflicts...

# Children auto-update with resolution!
```

**When to use**:
- Complex rebases with many conflicts
- Want to preserve context before resolving
- Team review of conflict resolution approach
- Conflicts in low-priority changes

### Advanced: Multi-Way Conflicts

Jj handles N-way conflicts (merging 3+ changes):

```bash
# Create 3-way merge
jj new change-a change-b change-c -m "Mega merge"

# Conflict markers show multiple diffs
cat conflicted.txt
# Shows snapshot + multiple diffs to apply
```

**Resolution strategy**:
- Apply each diff to snapshot one by one
- Or use conflict resolution tool with multi-way support

### Conflict Resolution Workflow Patterns

#### Pattern 1: Resolve Then Squash

```bash
# Hit conflict during rebase
jj new <conflicted-commit> -m "Resolve conflicts"
# Fix conflicts...

# Review resolution
jj diff

# Squash resolution into original
jj squash --into <conflicted-commit>
```

#### Pattern 2: Direct Edit

```bash
# Edit the conflicted commit directly
jj edit <conflicted-commit>
# Resolve conflicts...

# Create new working copy commit
jj new
```

**Choose based on**:
- Use Pattern 1 when you want to review resolution as a diff
- Use Pattern 2 for quick, obvious resolutions

### Aborting/Undoing Conflict Operations

```bash
# Undo last operation (including one that created conflicts)
jj undo

# Or restore to earlier operation
jj op log  # Find operation before conflict
jj op restore <operation-id>

# Abandon conflicted commit entirely
jj abandon <conflicted-commit>
# Children will rebase to its parent
```

---

## History Editing

### Rewriting Commit Messages

```bash
# Edit any commit's description
jj describe -r <commit> -m "New description"

# Or use editor
jj describe -r <commit>
# Opens $EDITOR

# Edit working copy commit description
jj describe -m "Working copy description"
```

**Key difference from Git**: No interactive rebase needed!

### Splitting Commits

```bash
# Split current commit interactively
jj split -i
# Choose which hunks go in first vs second commit

# Split specific commit
jj split -r <commit> -i

# Split in non-interactive scripts
# Create sibling with some files
jj squash --from <commit> --into <parent> path/to/file.txt
```

**Use cases**:
- Separate unrelated changes
- Make commits more reviewable
- Fix overly large commits

### Combining Commits

```bash
# Squash commit into its parent
jj squash -r <commit>

# Squash interactively (choose what to squash)
jj squash -r <commit> -i

# Squash into non-parent
jj squash --into <target-commit>

# Path-limited squash into another commit between commits
jj squash --from <source> --into <dest> path/to/file.txt
```

### Reordering Commits

```bash
# Rebase commit to new parent
jj rebase -r <commit> -d <new-parent>

# Rebase entire stack
jj rebase -s <bottom> -d <new-parent>

# Insert new commit in middle of stack
jj new -A <commit>  # After this commit
# or
jj new -B <commit>  # Before this commit
```

**Example: Swap two commits**:
```bash
# Before: A -> B -> C
# Want: A -> C -> B

jj rebase -r B -d C
# Now: A -> C -> B
```

### Moving Changes Between Commits

```bash
# Move changes from commit to its parent
jj squash -r <commit>

# Move changes interactively
jj squash -r <commit> -i

# Path-limited squash into another commit
jj squash -r <commit> path/to/file.txt

# Move to arbitrary commit (not just parent)
jj squash --from <source> --into <dest>
jj squash --from @ --into <commit> -i  # Interactive
```

### Editing Arbitrary Commits

```bash
# Edit commit directly in working copy
jj edit <commit>
# Make changes...
# Changes automatically amend the commit

# Return to tip of branch
jj new <branch-tip>

# Or create new commit after editing
jj new
```

### Duplicating Changes

```bash
# Duplicate a commit
jj duplicate <commit>

# Duplicate and place elsewhere
jj duplicate <commit>
jj rebase -r <duplicated-id> -d <target>
```

**Use cases**:
- Cherry-pick equivalent
- Experimenting with variations
- Applying fix to multiple branches

### Abandoning Commits

```bash
# Remove commit from history
jj abandon <commit>
# Children rebase onto parent

# Abandon multiple commits
jj abandon <commit1> <commit2> <commit3>

# Abandon range
jj abandon <start>::<end>
```

**Difference from Git**: Descendants are automatically rebased, not orphaned!

### Safe History Rewriting

Everything is tracked in the operation log:

```bash
# View all operations
jj op log

# Undo last operation (repeat to go further back)
jj undo
jj undo   # again → older still

# Restore to a specific operation id from `jj op log`
jj op restore <operation-id>

# View state at a previous operation (global flag)
jj --at-operation <operation-id> log
# short alias:
jj --at-op <operation-id> log
```

**Best practices**:
1. Check `jj op log` before major operations
2. Use `jj undo` liberally - it's safe
3. Back up repo before bulk rewrites (though op log usually saves you)

---

## Large Repository Strategies

### Sparse Checkouts

```bash
# Start with minimal working copy
jj sparse set --clear --add src/

# Add more paths
jj sparse set --add tests/
jj sparse set --add docs/

# List current sparse patterns
jj sparse list

# Reset to full checkout
jj sparse reset
```

**Use cases**:
- Monorepos
- Repos with large binary assets
- Working on specific subsystem

### Multiple Workspaces

Alternative to git worktree:

```bash
# Create additional workspace
jj workspace add ../repo-workspace-2

# Each workspace can check out different commit
cd ../repo-workspace-2
jj edit <different-commit>

# Work independently in each
# Commits are shared, working copies are separate

# List workspaces
jj workspace list

# Remove workspace when done
jj workspace forget workspace-2
rm -rf ../repo-workspace-2
```

**Advantages over git worktree**:
- Cleaner mental model
- Better integrated with jj's design
- Automatic operation log sharing

**Use cases**:
- Long-running builds in one workspace
- Code review in separate workspace
- Testing different branches simultaneously

### Efficient Fetching

```bash
# Fetch only what you need
jj git fetch --remote origin

# Fetch specific bookmarks
jj git fetch --bookmark main --bookmark develop

# Background fetch (if your workflow allows)
# Set up as cron job or Git hook
jj git fetch --all-remotes
```

### Performance Tips

1. **Configure revset for faster log**:
```toml
# ~/.config/jj/config.toml
[ui]
default-revset = "@::@ | (main..@) | trunk()"  # Show only relevant commits
```

2. **Limit log output**:
```bash
jj log --limit 20
jj log -r 'ancestors(@, 5)'  # Only 5 ancestors
```

3. **Pack Git refs periodically**:
```bash
jj util gc  # Packs Git refs, speeds up import
```

4. **Use snapshot.auto-track selectively**:
```toml
[snapshot]
auto-track = "none()"  # Don't auto-track new files
# Then manually track: jj file track <file>
```

### Monorepo Patterns

```bash
# Work on specific project
cd monorepo
jj sparse set --clear --add projects/my-project/

# Create changes scoped to project
jj new -m "my-project: Add feature X"

# Multiple people in monorepo
# Use clear commit message prefixes
jj new -m "frontend: Update React version"
jj new -m "backend: Add API endpoint"
jj new -m "infra: Update deployment config"
```

---

## CI/CD Integration

### Basic CI Setup

Jj works transparently with Git-based CI:

```bash
# In CI script
jj git clone <repo-url>
cd repo

# Run tests on specific commit
jj new -r <commit-to-test>
./run-tests.sh

# CI sees normal Git repo
# Can use git commands if needed
```

### Testing Stacked Changes

```bash
# CI can test each level of stack
for commit in $(jj log -r 'feature-stack::' --no-graph -T 'commit_id'); do
    jj new -r $commit
    ./run-tests.sh || exit 1
done
```

### Bookmark-Based CI

```yaml
# GitHub Actions example
on:
  push:
    branches:
      - main
      - 'feature/**'

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Run tests
        run: ./test.sh
```

Works identically whether you push with `git` or `jj`!

### Auto-Rebase in CI

```bash
# CI script to keep feature branches updated
jj git fetch
jj rebase -r feature-bookmark -d main@origin
jj git push --bookmark feature-bookmark --force
```

### Pre-push Hooks

Jj doesn't support Git hooks directly, but you can:

```bash
# Wrapper script: ~/bin/jj-push
#!/bin/bash
./run-tests.sh || exit 1
jj git push "$@"

# Use in workflow
jj bookmark set my-feature
jj-push --bookmark my-feature
```

### Commit Signing in CI

```bash
# Configure signing
jj config set --user signing.sign-all true
jj config set --user signing.backend gpg
jj config set --user signing.key <key-id>

# Or use SSH signing
jj config set --user signing.backend ssh
jj config set --user signing.key ~/.ssh/id_ed25519
```

---

## Common Pitfalls & Gotchas

### 1. Bookmarks Don't Follow Commits

**Problem**: Created a bookmark, made new commits, bookmark still at old commit.

```bash
# Wrong expectation (from Git)
jj bookmark create feature
jj new -m "New work"
jj log  # Bookmark is NOT at new commit!

# Correct approach
jj bookmark create feature
jj new -m "New work"
jj bookmark set feature  # Explicitly move bookmark
```

**Solution**: Remember bookmarks are pointers you manually move, or use `jj git push --change @` to auto-create.

### 2. Divergent Changes

**Problem**: Same change ID points to multiple commits.

```bash
jj log
# Shows: mychange?? or mychange/1, mychange/2

# Happens when:
# - Mixing jj and git commands
# - Concurrent operations
# - Import/export sync issues
```

**Solution**:
```bash
# Resolve by choosing which one is correct
jj bookmark list  # See divergent commits
jj bookmark set feature -r <correct-commit>

# Or abandon unwanted version
jj abandon <unwanted-commit>
```

### 3. Empty Merge Commits

**Problem**: Merge commits show as "(empty)" in log.

**Explanation**: Jj defines changes as diff from auto-merged parents. Clean merges have no additional changes, thus "empty".

**This is normal!** Only non-empty merges contain conflict resolutions or extra changes.

### 4. Working Copy Conflicts

**Problem**: Tools complain about weird file contents.

**Cause**: Working copy contains conflict markers. Git tools don't understand jj's conflict representation.

**Solution**:
- Resolve conflicts before using Git tools
- Or work in non-conflicted commits
- Use `jj new <non-conflicted>` to get clean working copy

### 5. Lost Changes

**Problem**: "I ran a command and my work disappeared!"

**Solution**: Check operation log:
```bash
jj op log  # Find operation before loss
jj op restore <operation-id>
# Or
jj undo
```

**Actual data loss is extremely rare** - almost always recoverable via operation log.

### 6. Forgetting to Push

**Problem**: Made changes, collaborator doesn't see them.

**Remember**:
```bash
jj bookmark set feature  # Make sure bookmark is updated
jj git push --bookmark feature  # Push to remote
```

Unlike Git, jj doesn't have a "current branch" that auto-pushes.

### 7. Colocated Repo Confusion

**Problem**: Mixing jj and git creates confusion.

**Best Practice**:
- Primarily use jj commands
- Use git only for read operations
- Avoid mutating git commands
- Use `jj git import/export` if needed

### 8. Conflict Markers in Git View

**Problem**: Git tools show strange content for conflicted commits.

**Why**: Jj stores conflicts in non-human-readable format; materializes as markers in working copy.

**Solution**: Don't use Git tools on conflicted commits, or resolve conflicts first.

### 9. Slow Operations in Large Repos

**Problem**: Commands feel slow.

**Solutions**:
```bash
# Pack Git refs
jj util gc

# Use sparse checkout
jj sparse set --clear --add <paths>

# Limit default log view
jj config set --user ui.default-revset '@::@ | trunk()'
```

### 10. Auto-tracking Unwanted Files

**Problem**: Temporary/build files get tracked.

**Solutions**:
```bash
# Add to .gitignore FIRST
echo "*.tmp" >> .gitignore

# Then untrack
jj file untrack "*.tmp"

# Or configure selective auto-track
jj config set --user snapshot.auto-track 'glob:"src/**"'
```

---

## Migration Guide

### From Git to Jujutsu

#### Week 1: Learn the Basics

```bash
# Install jj
brew install jj  # or your package manager

# Try it on a test repo
git clone <test-repo>
cd test-repo
jj git init --git-repo .

# Learn core commands
jj log
jj status
jj new -m "Test change"
jj diff
jj describe
```

**Goal**: Understand working copy commit, changes vs revisions, automatic snapshotting.

#### Week 2: Replace Daily Git Commands

| Git Command | Jujutsu Equivalent | Notes |
|-------------|-------------------|-------|
| `git status` | `jj status` | Shows working copy changes |
| `git log` | `jj log` | More informative by default |
| `git diff` | `jj diff` | Diff current changes |
| `git add .` | Automatic | Working copy auto-tracked |
| `git commit -m` | `jj describe -m` + `jj new` | Or just `jj new -m` |
| `git commit --amend` | `jj squash` | Default amends parent |
| `git checkout <branch>` | `jj edit <commit>` | Edit commit directly |
| `git checkout -b <branch>` | `jj new -m` | Create change, bookmark later |
| `git merge` | `jj new <c1> <c2>` | Creates merge commit |
| `git rebase -i` | Multiple commands | `jj rebase`, `jj split`, `jj squash` |
| `git push` | `jj git push --bookmark <name>` | Must specify bookmark |
| `git fetch` | `jj git fetch` | Same |
| `git pull` | `jj git fetch` + `jj rebase` | Separate operations |

#### Week 3: Adopt Advanced Workflows

```bash
# Use stacked changes
jj new main -m "Step 1"
jj new -m "Step 2"
jj new -m "Step 3"

# Interactive splitting
jj split -i

# Conflict management
jj rebase -r <change> -d main
# Leave conflicts, keep working
jj new -m "Build on conflicted"
```

#### Week 4: Configure Your Environment

```toml
# ~/.config/jj/config.toml

[user]
name = "Your Name"
email = "you@example.com"

[ui]
default-command = "log"
diff.format = "git"  # Or "color-words"

[ui]
pager = "less -FRX"

[template-aliases]
'format_short_id(id)' = 'id.shortest()'

[revset-aliases]
'mine()' = 'author(exact:"your@email.com")'
```

### Common Migration Questions

**Q: Do I need to migrate my whole team?**
A: No! Jj is Git-compatible. You can use jj while teammates use git.

**Q: What about existing branches?**
A: They appear as bookmarks in jj. Work with them normally.

**Q: Can I go back to Git?**
A: Yes, anytime. It's just a Git repo. Remove `.jj/` directory if needed.

**Q: What about Git GUIs?**
A: They work on the Git repo. Mix carefully (see colocated repos section).

### Gradual Adoption Strategy

1. **Phase 1: Solo Projects** (Month 1)
   - Use jj on personal repos
   - Get comfortable with core commands
   - Learn operation log safety net

2. **Phase 2: Read-only at Work** (Month 2)
   - Clone work repos with jj
   - Use jj to browse history, review code
   - Still push/commit with git if unsure

3. **Phase 3: Full Personal Workflow** (Month 3)
   - Use jj for all personal development
   - Push to work repos with jj
   - Teammates don't notice difference

4. **Phase 4: Advocate** (Ongoing)
   - Share workflows with interested teammates
   - Help others adopt gradually
   - Contribute to jj ecosystem

### Unlearning Git Habits

**Stop thinking about**:
- Staging area (no index in jj)
- "Current branch" (use bookmarks intentionally)
- Detached HEAD warnings (jj's normal state)
- Rebase conflicts as blockers (conflicts are first-class)

**Start thinking about**:
- Working copy as a commit
- Changes that evolve
- Revsets as powerful queries
- Operation log as safety net

---

## Additional Resources

### Official Documentation
- [Jujutsu Tutorial](https://martinvonz.github.io/jj/latest/tutorial/)
- [Jujutsu Documentation](https://martinvonz.github.io/jj/latest/)

### Community Resources
- [GitHub Repository](https://github.com/jj-vcs/jj)
- [Discord Community](https://discord.gg/dkmfj3aGQN)

### Blog Posts & Guides
- [Chris Krycho's "jj init"](https://v5.chriskrycho.com/essays/jj-init/) - Comprehensive introduction
- [Steve Klabnik's Tutorial](https://steveklabnik.github.io/jujutsu-tutorial/) - Alternative tutorial

### Tools & Integrations
- [scm-diff-editor](https://github.com/arxanas/scm-record) - TUI diff editor
- Various GUIs listed in [jj wiki](https://github.com/jj-vcs/jj/wiki)

---

## Quick Reference Card

### Essential Commands

```bash
# Daily workflow
jj status                    # Check working copy
jj diff                      # See changes
jj log                       # View history
jj describe -m "msg"         # Describe current change
jj new -m "msg"              # Start new change

# Navigating history
jj edit <commit>             # Edit specific commit
jj new <commit>              # New change based on commit

# Sync with remote
jj git fetch                 # Get updates
jj git push --bookmark <br>  # Push bookmark
jj git push --change @       # Quick push with auto-bookmark

# Fixing mistakes
jj undo                      # Undo last operation
jj op log                    # View operation history
jj op restore <op>           # Restore to operation

# History editing
jj split -i                  # Split commit interactively
jj squash                    # Squash into parent
jj rebase -r <r> -d <dest>   # Rebase revision
jj abandon <commit>          # Remove commit

# Bookmarks
jj bookmark create <name>    # Create bookmark
jj bookmark set <name>       # Move bookmark to @
jj bookmark list             # List bookmarks

# Conflicts
jj resolve                   # Resolve with merge tool
# Edit conflict markers manually, then jj status
```

### Revset Patterns

```bash
jj log -r '@'                # Current commit
jj log -r 'main'             # Main bookmark
jj log -r '@-'               # Parent of current
jj log -r 'main..'           # Changes since main
jj log -r 'all()'            # Everything
jj log -r 'mine()'           # Your commits (with alias)
jj log -r '@::@'             # Just current commit
jj log -r 'main..@'          # From main to current
```

---

**Last Updated**: December 2024
**Jujutsu Version**: 0.23+ (check with `jj --version`)

---

*This guide is a living document. Contributions and corrections welcome!*
