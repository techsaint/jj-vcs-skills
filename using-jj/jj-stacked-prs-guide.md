# Stacked Pull Requests with Jujutsu (jj)

## Overview

This guide covers a comprehensive bookmark-based workflow for creating and managing stacked pull requests (PRs) in jj. Stacked PRs allow you to break large features into multiple reviewable chunks while maintaining linear dependencies between them.

**Key Principles:**
- Use bookmarks to organize a stack of related changes
- Maintain linear history from root to dev (head of stack)
- Create new commits and squash them into appropriate positions
- Handle external changes from collaborators
- Keep PRs small, focused, and independently reviewable

## Table of Contents

1. [Stack Structure and Naming](#stack-structure-and-naming)
2. [Initial Stack Setup](#initial-stack-setup)
3. [Creating the Stack](#creating-the-stack)
4. [Adding Work to Existing Stack Layers](#adding-work-to-existing-stack-layers)
5. [Common Workflows](#common-workflows)
6. [Handling External Changes](#handling-external-changes)
7. [Conflict Management](#conflict-management)
8. [Publishing and Syncing](#publishing-and-syncing)
9. [Stack Maintenance](#stack-maintenance)
10. [Troubleshooting](#troubleshooting)
11. [Best Practices](#best-practices)
12. [Advanced Techniques](#advanced-techniques)

---

## Stack Structure and Naming

### Bookmark Hierarchy

A well-organized stack uses a consistent naming convention:

```
TASK-ID/dev              ← Head of stack (latest work)
    ↑
TASK-ID/03-add-tests
    ↑
TASK-ID/02-implement-feature
    ↑
TASK-ID/01-add-types
    ↑
TASK-ID/root             ← Base of stack (usually main/master)
```

### Naming Convention

**Format:** `TASK-ID/NN-descriptive-title`

- **TASK-ID**: Ticket/issue identifier (e.g., `PROJ-1234`, `feat-auth`) look at existing branches to determine right format.
- **NN**: Sequential number (01, 02, 03...) indicating layer order
- **descriptive-title**: Short kebab-case description of what this layer does

**Examples:**
```
AUTH-42/root
AUTH-42/01-database-schema
AUTH-42/02-api-endpoints
AUTH-42/03-frontend-integration
AUTH-42/04-tests
AUTH-42/dev
```

### Why This Structure?

1. **Linear history**: Each bookmark builds on the previous one
2. **Clear dependencies**: Numbering shows the order
3. **Easy navigation**: Descriptive names explain each layer
4. **Group related work**: TASK-ID prefix groups all stack bookmarks
5. **Dedicated dev branch**: `/dev` is the working head, other branches are stable PR targets

---

## Initial Stack Setup

### Step 1: Create Root Bookmark

Start by creating a root bookmark from your base branch (usually `main` or `master`):

```bash
# Ensure you're on the latest main
jj git fetch
jj new main -m "Start AUTH-42 stack"

# Create the root bookmark
jj bookmark create AUTH-42/root
```

**What this does:**
- Creates a new change based on `main`
- Establishes the foundation for your stack
- Root bookmark marks the divergence point from main

### Step 2: Plan Your Stack

Before creating bookmarks, plan your layers:

1. **Identify logical separations** - Database changes, API, UI, tests
2. **Consider review size** - Each layer should be reviewable in one sitting
3. **Think about dependencies** - Each layer should build on the previous
4. **Plan for iterations** - Leave room for feedback incorporation

NOTE: if there's already work been done in the current branch, load [jj-convert-to-stacked-prs.md]( ./jj-convert-to-stacked-prs.md).

**Example Planning:**
```
Layer 01: Add database migrations and models
Layer 02: Implement core business logic
Layer 03: Add API endpoints
Layer 04: Add tests
```

### Step 3: Create Initial Bookmark Structure

Create all bookmarks upfront (you'll add commits later):

```bash
# Create first layer bookmark
jj new AUTH-42/root -m "Add database schema"
jj bookmark create AUTH-42/01-database-schema

# Create second layer bookmark
jj new AUTH-42/01-database-schema -m "Implement core logic"
jj bookmark create AUTH-42/02-core-logic

# Create third layer bookmark
jj new AUTH-42/02-core-logic -m "Add API endpoints"
jj bookmark create AUTH-42/03-api-endpoints

# Create dev bookmark (head of stack)
jj new AUTH-42/03-api-endpoints -m "WIP"
jj bookmark create AUTH-42/dev
```

**Verify your stack:**
```bash
jj log -r 'AUTH-42/root::AUTH-42/dev'
```

You should see a linear chain of empty or placeholder commits.

---

## Creating the Stack

### Understanding the Workflow

The workflow uses two key commands:
1. **`jj new -B BOOKMARK --no-edit`** - Create a new working commit at a specific bookmark
2. **`jj squash --into BOOKMARK`** - Squash your work into the target bookmark

**Why this approach?**
- **Flexibility**: Work anywhere in the stack without disrupting other layers
- **Safety**: Original bookmarks stay stable while you work
- **Precision**: Choose exactly where work belongs
- **Clean history**: Squashing keeps commits organized

### Working on Layer 01

```bash
# Start work on the first layer
jj new -B AUTH-42/01-database-schema --no-edit

# Make your changes
# Edit files: migrations/001_add_users.sql, models/user.py

# Check what you've done
jj status
jj diff

# Describe your work
jj describe -m "Add user table migration and User model"

# Squash this work into the 01 bookmark
jj squash --into AUTH-42/01-database-schema
```

**What happened:**
1. Created a new change positioned at the `01-database-schema` bookmark
2. Made changes to files
3. Described the work
4. Squashed it into the bookmark, keeping history clean

**View your stack:**
```bash
jj log -r 'AUTH-42/root::AUTH-42/dev'
```

You'll see the `01-database-schema` bookmark now has your actual changes.

### Working on Layer 02

```bash
# Start work on the second layer
jj new -B AUTH-42/02-core-logic --no-edit

# Make your changes
# Edit files: services/auth_service.py

jj describe -m "Implement password hashing and validation"
jj squash --into AUTH-42/02-core-logic
```

### Working on Layer 03

```bash
# Start work on the third layer
jj new -B AUTH-42/03-api-endpoints --no-edit

# Make your changes
# Edit files: api/routes.py, api/schemas.py

jj describe -m "Add /login and /register endpoints"
jj squash --into AUTH-42/03-api-endpoints
```

### Working on Dev (Experimental Work)

The `/dev` bookmark is your playground for work-in-progress:

```bash
# Work on dev
jj new -B AUTH-42/dev --no-edit

# Experiment with changes
# Edit files: dashboard.py

jj describe -m "WIP: Add user dashboard"
jj squash --into AUTH-42/dev
```

**Dev bookmark is special:**
- Not meant for PRs (unless it becomes a layer)
- Can be messy
- Can be rebased aggressively
- Once work stabilizes, promote it to a numbered layer

---

## Adding Work to Existing Stack Layers

### The Key Challenge

When you need to add more work to a layer that already exists, you must:
1. Position your new change correctly
2. Avoid conflicts with descendant layers
3. Maintain linear history

### Method 1: New Change + Squash (Recommended)

This is the standard approach for adding work to any layer:

```bash
# Add more work to layer 01
jj new -B AUTH-42/01-database-schema --no-edit

# Make changes
# Edit files: migrations/002_add_sessions.sql

jj describe -m "Add sessions table"
jj squash --into AUTH-42/01-database-schema
```

**What happens:**
- New change created at the bookmark
- Changes made
- Squashed into bookmark
- All descendants (02, 03, dev) automatically rebased!

### Method 2: Edit in Place (Advanced)

For small tweaks, you can edit the bookmark commit directly:

```bash
# Edit the bookmark commit directly
jj edit AUTH-42/01-database-schema

# Make changes
# Edit files: migrations/001_add_users.sql (fix typo)

# Create a new change to continue working
jj new -m "Continue work"
```

**Warning:** This changes the commit, which:
- Triggers automatic rebase of descendants
- Can cause conflicts if descendants touch the same files
- Is more error-prone than squashing

**Use Method 1 (squash) for most work.**

### Choosing the Right Position

**Where should you add work?**

```
Goal: Add logging to the auth service
Question: Which layer?

AUTH-42/01-database-schema    ← No, logging isn't database-related
AUTH-42/02-core-logic         ← YES! Logging is part of core logic
AUTH-42/03-api-endpoints      ← No, logging belongs in the service layer
```

**Guidelines:**
1. Add work to the earliest layer where it logically belongs
2. If work spans multiple layers, split it
3. If unsure, add to current layer and refactor later

### Adding Work That Spans Multiple Layers

Sometimes new work affects multiple layers:

```bash
# Scenario: Need to add email validation
# Layer 01: Add email column to database
jj new -B AUTH-42/01-database-schema --no-edit
# Edit: migrations/003_add_email.sql
jj describe -m "Add email column to users table"
jj squash --into AUTH-42/01-database-schema

# Layer 02: Add email validation logic
jj new -B AUTH-42/02-core-logic --no-edit
# Edit: services/auth_service.py (add validate_email)
jj describe -m "Add email validation"
jj squash --into AUTH-42/02-core-logic

# Layer 03: Update API to accept email
jj new -B AUTH-42/03-api-endpoints --no-edit
# Edit: api/schemas.py (add email field)
jj describe -m "Add email field to registration endpoint"
jj squash --into AUTH-42/03-api-endpoints
```

**Each layer gets its relevant changes:**
- Database schema in layer 01
- Business logic in layer 02
- API interface in layer 03

---

## Common Workflows

### Workflow 1: Iterating on a Layer

You'll often revisit layers based on review feedback:

```bash
# Get feedback: "Add index to user email in database"
jj new -B AUTH-42/01-database-schema --no-edit

# Add the index
# Edit: migrations/001_add_users.sql

jj describe -m "Add index on user.email"
jj squash --into AUTH-42/01-database-schema

# Push updated changes
jj git push --bookmark AUTH-42/01-database-schema
```

### Workflow 2: Promoting Dev Work to a Layer

Once work in `/dev` stabilizes, promote it to a proper layer:

```bash
# Dev work is ready to become layer 04
jj new AUTH-42/03-api-endpoints -m "Add comprehensive tests"
jj bookmark create AUTH-42/04-tests

# Move work from dev to new layer
jj new -B AUTH-42/04-tests --no-edit
# Copy/move your test files from dev

jj describe -m "Add unit and integration tests"
jj squash --into AUTH-42/04-tests

# Update dev to point to new layer
jj bookmark set AUTH-42/dev --revision AUTH-42/04-tests
```

### Workflow 3: Inserting a New Layer

Sometimes you need to insert a layer in the middle:

```bash
# Current stack:
# 01-database-schema
# 02-api-endpoints (too big!)
# dev

# Need to split: create 02-core-logic between 01 and original 02

# Step 1: Rename original 02 to 03
jj bookmark rename AUTH-42/02-api-endpoints AUTH-42/03-api-endpoints

# Step 2: Create new layer 02
jj new AUTH-42/01-database-schema -m "Core business logic"
jj bookmark create AUTH-42/02-core-logic

# Step 3: Move appropriate changes from 03 to 02
jj new -B AUTH-42/02-core-logic --no-edit
# Extract core logic that was in layer 03
# Edit: services/auth_service.py

jj squash --into AUTH-42/02-core-logic

# Step 4: Update layer 03 to build on 02
jj rebase -s AUTH-42/03-api-endpoints -d AUTH-42/02-core-logic
```

### Workflow 4: Viewing Stack Status

```bash
# See all bookmarks in your stack
jj bookmark list | grep AUTH-42

# See commits in linear order
jj log -r 'AUTH-42/root::AUTH-42/dev'

# See specific layer
jj show AUTH-42/02-core-logic

# See what changed in a layer
jj diff -r AUTH-42/01-database-schema

# Compare two layers
jj diff --from AUTH-42/01-database-schema --to AUTH-42/02-core-logic
```

---

## Handling External Changes

### Scenario 1: Someone Pushed to Your PR Branch

A collaborator added commits to one of your PR branches on GitHub:

```bash
# Fetch latest changes
jj git fetch

# Check what changed
jj log -r 'AUTH-42/02-core-logic'

# Your bookmark is now behind the remote
# GitHub shows: AUTH-42/02-core-logic has 2 new commits

# Option A: Rebase your bookmark onto the remote (clean history)
jj rebase -s AUTH-42/02-core-logic -d AUTH-42/02-core-logic@origin

# Option B: Merge remote changes (preserves both histories)
jj new -B AUTH-42/02-core-logic AUTH-42/02-core-logic@origin --no-edit
jj squash --into AUTH-42/02-core-logic
```

**Recommended: Option A (rebase)** for cleaner history.

### Scenario 2: External Changes Conflict with Your Work

```bash
# Fetch changes
jj git fetch

# Try to rebase
jj rebase -s AUTH-42/02-core-logic -d AUTH-42/02-core-logic@origin

# Conflicts occur!
# jj will show: "Conflict in services/auth_service.py"

# Resolve conflicts
jj status  # Shows conflicting files
jj diff    # Shows conflict markers

# Edit files to resolve
# Edit: services/auth_service.py (remove conflict markers)

# Check resolution
jj diff

# Continue (automatic - jj commits resolved conflicts)
jj new -m "Continue after resolving conflicts"
```

### Scenario 3: Syncing External Changes Through the Stack

When layer 02 gets external changes, layers 03 and dev need updating:

```bash
# Someone pushed to layer 02
jj git fetch

# Update layer 02
jj rebase -s AUTH-42/02-core-logic -d AUTH-42/02-core-logic@origin

# Descendants (03, dev) are automatically rebased!
# Check the entire stack
jj log -r 'AUTH-42/root::AUTH-42/dev'
```

**jj's automatic rebasing saves you!**
- All descendant commits automatically rebase
- Conflicts are detected immediately
- You only fix conflicts once

### Scenario 4: Main Branch Advanced

Your stack's root needs updating when `main` moves forward:

```bash
# Fetch latest main
jj git fetch

# Update root to latest main
jj bookmark set AUTH-42/root --revision main

# Rebase entire stack onto new root
jj rebase -s 'AUTH-42/01-database-schema' -d AUTH-42/root

# All layers (01, 02, 03, dev) now rebased!
```

**When to update root:**
- Before publishing PRs (ensures compatibility)
- When main has changes your stack needs
- When preparing to merge

---

## Conflict Management

### Understanding Conflicts in Stacked PRs

Conflicts can occur at multiple levels:

1. **Within a layer**: Your changes conflict with each other
2. **Between layers**: Layer 03 conflicts with changes in layer 02
3. **With external changes**: Remote changes conflict with your work
4. **With main**: Stack root conflicts with updated main

### Minimizing Conflicts Through Positioning

**Key principle: Add changes to the earliest relevant layer**

```bash
# BAD: Adding database changes to layer 03
jj new -B AUTH-42/03-api-endpoints --no-edit
# Edit: migrations/add_column.sql  ← Database work in API layer!
jj squash --into AUTH-42/03-api-endpoints
# This will conflict if layer 01 or 02 later touches the same tables

# GOOD: Adding database changes to layer 01
jj new -B AUTH-42/01-database-schema --no-edit
# Edit: migrations/add_column.sql  ← Database work in database layer
jj squash --into AUTH-42/01-database-schema
# Layer 02 and 03 automatically rebase - clean!
```

**Why this matters:**
- Changes in layer 01 propagate forward cleanly
- Changes in layer 03 can conflict with earlier layers
- Working "upstream" (earlier layers) is safer

### Resolving Conflicts Step-by-Step

```bash
# Conflict detected during squash
jj new -B AUTH-42/02-core-logic --no-edit
# Edit: services/auth_service.py
jj squash --into AUTH-42/02-core-logic
# Error: Conflict in services/auth_service.py

# Step 1: Check conflict status
jj status
# Shows: services/auth_service.py has conflicts

# Step 2: View the conflict
jj diff
# Shows conflict markers:
# <<<<<<< left
# Your changes
# =======
# Existing changes
# >>>>>>> right

# Step 3: Edit the file to resolve
# Edit: services/auth_service.py
# Remove conflict markers, keep desired changes

# Step 4: Verify resolution
jj diff  # Should show no conflict markers
jj status  # Should show clean state

# Step 5: Continue working
jj new -m "Continue after conflict resolution"
```

### Advanced Conflict Resolution

**Using external merge tools:**

```bash
# Configure merge tool (one-time setup)
jj config set --user ui.merge-tool meld

# When conflict occurs
jj resolve  # Opens merge tool

# Or resolve specific file
jj resolve services/auth_service.py
```

**Understanding jj's conflict markers:**

```python
 <<<<<<< left (your changes)
def authenticate(password):
    return bcrypt.verify(password)
 =======
def authenticate(username, password):
    return verify_password(password)
 >>>>>>> right (existing code)
```

**Resolution strategies:**
1. **Keep left**: Your new changes are correct
2. **Keep right**: Existing code is correct
3. **Merge both**: Combine both sets of changes
4. **Rewrite**: Neither is correct, write new code

### Preventing Conflicts

**Best practices:**

1. **Keep layers focused** - Each layer touches different files
2. **Work upstream** - Add changes to earliest relevant layer
3. **Fetch often** - Stay synchronized with remote
4. **Small commits** - Easier to resolve conflicts
5. **Review diffs** - Check what descendants might conflict

**Example of conflict-prone structure:**
```
Layer 01: Add User model
Layer 02: Modify User model  ← Will conflict!
Layer 03: Add User methods   ← Will conflict with 01 and 02!
```

**Better structure:**
```
Layer 01: Complete User model (all fields, all methods)
Layer 02: Add AuthService (uses User)
Layer 03: Add API endpoints (uses AuthService)
```

---

## Publishing and Syncing

### Publishing Individual Layers as PRs

Each numbered bookmark becomes a separate PR:

```bash
# Publish layer 01
jj git push --bookmark AUTH-42/01-database-schema

# On GitHub: Create PR
# Title: "[AUTH-42][1/4] Add database schema"
# Base: main (or AUTH-42/root if it differs)
# Description: First layer of authentication feature stack
```

**PR naming convention:**
- Title: `[TASK-ID][N/Total] Layer description`
- Example: `[AUTH-42][2/4] Implement core authentication logic`

### Publishing Dependent Layers

Layer 02 depends on layer 01:

```bash
# Publish layer 02
jj git push --bookmark AUTH-42/02-core-logic

# On GitHub: Create PR
# Title: "[AUTH-42][2/4] Implement core authentication logic"
# Base: AUTH-42/01-database-schema  ← Not main!
# Description:
#   Second layer of authentication feature stack.
#   Depends on: #1234 (link to layer 01 PR)
```

**Key: Set base branch to previous layer!**

### Publishing the Entire Stack

```bash
# Push all bookmarks at once
jj git push --bookmark 'glob:AUTH-42/*'

# Or push individually
jj git push --bookmark AUTH-42/01-database-schema
jj git push --bookmark AUTH-42/02-core-logic
jj git push --bookmark AUTH-42/03-api-endpoints
```

**On GitHub:**
1. Create PR for layer 01 → base: main
2. Create PR for layer 02 → base: AUTH-42/01-database-schema
3. Create PR for layer 03 → base: AUTH-42/02-core-logic

**Result:**
- 3 separate PRs
- Each reviewable independently
- Clear dependency chain

### Syncing After Remote Changes

```bash
# Regular sync workflow
jj git fetch  # Get remote changes

# Check what changed
jj log -r 'AUTH-42/root::AUTH-42/dev'

# If remote bookmark moved, update local
jj bookmark set AUTH-42/01-database-schema --revision AUTH-42/01-database-schema@origin

# Or rebase if you have local unpushed changes
jj rebase -s AUTH-42/01-database-schema -d AUTH-42/01-database-schema@origin
```

### Force Push Workflow

After rebasing or squashing, you'll need to force push:

```bash
# You've squashed new changes into layer 02
jj new -B AUTH-42/02-core-logic --no-edit
# Make changes
jj squash --into AUTH-42/02-core-logic

# Push with force (rewrites remote)
jj git push --bookmark AUTH-42/02-core-logic --force

# Warning: Coordinate with collaborators!
```

**When to force push:**
- After squashing into a bookmark
- After rebasing onto updated base
- After addressing review feedback

**When NOT to force push:**
- Someone else is working on the branch
- PR is approved (unless requested)
- You're unsure about the changes

---

## Stack Maintenance

### Keeping Stack in Sync

Regular maintenance keeps your stack healthy:

```bash
# Daily sync routine
jj git fetch                          # Get remote changes
jj bookmark set AUTH-42/root -r main  # Update root
jj rebase -s AUTH-42/01-database-schema -d AUTH-42/root  # Rebase stack

# Verify stack integrity
jj log -r 'AUTH-42/root::AUTH-42/dev'
```

### Cleaning Up Merged Layers

After a PR merges:

```bash
# Layer 01 merged to main
jj git fetch

# Delete local bookmark
jj bookmark delete AUTH-42/01-database-schema

# Update layer 02 to build on main instead
jj rebase -s AUTH-42/02-core-logic -d main

# Or update root and rebase remaining stack
jj bookmark set AUTH-42/root -r main
jj rebase -s AUTH-42/02-core-logic -d AUTH-42/root
```

### Collapsing Layers

Sometimes you want to merge two layers:

```bash
# Merge layer 02 into layer 01
# (Maybe they should have been one layer)

# Move changes from 02 into 01
jj rebase -s 'AUTH-42/02-core-logic' -d AUTH-42/01-database-schema
jj bookmark set AUTH-42/01-database-schema -r AUTH-42/02-core-logic
jj bookmark delete AUTH-42/02-core-logic

# Renumber remaining layers
jj bookmark rename AUTH-42/03-api-endpoints AUTH-42/02-api-endpoints
```

### Splitting a Layer

If a layer becomes too large:

```bash
# Layer 02 is too big, split it

# Create new intermediate layer
jj new AUTH-42/01-database-schema -m "Core validation logic"
jj bookmark create AUTH-42/02-validation

# Move some changes from old layer 02 (now will be 03)
jj new -B AUTH-42/02-validation --no-edit
# Cherry-pick or manually copy validation-related files
jj squash --into AUTH-42/02-validation

# Rename original layer 02
jj bookmark rename AUTH-42/02-core-logic AUTH-42/03-core-logic

# Rebase layer 03 onto new layer 02
jj rebase -s AUTH-42/03-core-logic -d AUTH-42/02-validation
```

---

## Troubleshooting

### Problem: Bookmark Doesn't Point Where I Think

```bash
# Check where bookmark points
jj log -r AUTH-42/02-core-logic

# Move bookmark to desired commit
jj bookmark set AUTH-42/02-core-logic --revision <commit-id>
```

### Problem: Accidentally Squashed Into Wrong Bookmark

```bash
# You squashed into layer 03 instead of layer 02
# Use operation log to undo

jj op log  # Find operation before squash
jj undo  # Undo last operation

# Or restore to specific operation
jj op restore <operation-id>
```

### Problem: Stack History Isn't Linear

```bash
# Check for non-linear history
jj log -r 'AUTH-42/root::AUTH-42/dev'

# See a merge or branch? Fix it:
jj rebase -s <non-linear-commit> -d <parent>
```

### Problem: Lost Track of Which Layer to Work On

```bash
# View all layers with commit counts
jj log -r 'AUTH-42/root::AUTH-42/dev' --summary

# See what files each layer touches
jj diff -r AUTH-42/01-database-schema --summary
jj diff -r AUTH-42/02-core-logic --summary

# Check current position
jj status
```

### Problem: Pushed Wrong Bookmark

```bash
# Pushed AUTH-42/dev to remote by accident
# Delete remote bookmark
jj git push --bookmark AUTH-42/dev --delete

# Or overwrite with correct version
jj bookmark set AUTH-42/dev --revision <correct-commit>
jj git push --bookmark AUTH-42/dev --force
```

### Problem: Merge Conflicts Everywhere After Rebase

```bash
# Too many conflicts after rebase
# Abort and try different approach

jj undo  # Undo the rebase

# Alternative: Rebase layer by layer
jj rebase -s AUTH-42/01-database-schema -d main
# Resolve conflicts
jj rebase -s AUTH-42/02-core-logic -d AUTH-42/01-database-schema
# Resolve conflicts
# Continue layer by layer
```

### Problem: Bookmark Tracking Wrong Remote

```bash
# Check bookmark tracking
jj bookmark list

# Set correct tracking
jj git fetch
jj bookmark set AUTH-42/01-database-schema --revision AUTH-42/01-database-schema@origin
```

---

## Best Practices

### 1. Plan Your Stack Before Creating

```bash
# Bad: Creating layers as you go, no plan
jj new -m "random work"  # What layer is this?

# Good: Plan first, create structure, fill in work
# 01: Database
# 02: Core logic
# 03: API
# 04: Tests
```

### 2. Keep Layers Focused and Independent

**Good layer separation:**
- Layer 01: Database schema only
- Layer 02: Business logic only
- Layer 03: API endpoints only

**Bad layer separation:**
- Layer 01: Database + some API + random tests
- Layer 02: More database + business logic
- Layer 03: Everything else

### 3. Use Descriptive Bookmark Names

```bash
# Bad names
AUTH-42/01-stuff
AUTH-42/02-more
AUTH-42/03-final

# Good names
AUTH-42/01-database-schema
AUTH-42/02-password-hashing
AUTH-42/03-login-endpoints
```

### 4. Work Upstream (Early Layers) When Possible

```bash
# Need to add a new database column
# Add it in layer 01, not layer 03!

jj new -B AUTH-42/01-database-schema --no-edit
# Add column
jj squash --into AUTH-42/01-database-schema
# Layers 02, 03 automatically rebase
```

### 5. Keep Dev Clean or Frequently Reset

```bash
# Option A: Keep dev as empty head
jj bookmark set AUTH-42/dev --revision AUTH-42/03-api-endpoints

# Option B: Use dev for experiments, reset often
jj bookmark set AUTH-42/dev --revision AUTH-42/03-api-endpoints
# Wipes out experimental work - be sure you want this!
```

### 6. Document Dependencies in PR Descriptions

```markdown
## PR Description Template

Title: [AUTH-42][2/4] Implement password hashing

**Stack Position:** 2 of 4
**Depends on:** #1234 (AUTH-42/01-database-schema)
**Depended on by:** #1236 (AUTH-42/03-login-endpoints)

This PR implements bcrypt password hashing...

### Review Notes
- Review #1234 first (database schema)
- This PR can be reviewed independently of #1236
```

### 7. Communicate Force Pushes

```bash
# Before force pushing to shared branch
# Notify collaborators in PR comments:

# "Force pushing to address review feedback.
#  If you're working on this branch, please sync:
#  jj git fetch
#  jj bookmark set AUTH-42/02-core-logic -r AUTH-42/02-core-logic@origin"

jj git push --bookmark AUTH-42/02-core-logic --force
```

### 8. Regular Stack Hygiene

```bash
# Weekly maintenance
jj git fetch                          # Sync remote
jj bookmark set AUTH-42/root -r main  # Update base
jj rebase -s AUTH-42/01-database-schema -d AUTH-42/root  # Rebase stack
jj log -r 'AUTH-42/root::AUTH-42/dev'  # Verify structure
```

### 9. Use Operation Log Liberally

```bash
# Before risky operation
jj op log  # Note current operation ID

# Do risky thing
jj rebase -s AUTH-42/02-core-logic -d AUTH-42/01-database-schema

# If it goes wrong
jj undo  # Or restore to noted operation
```

### 10. Delete Merged Bookmarks Promptly

```bash
# After PR merges
jj bookmark delete AUTH-42/01-database-schema
git push --delete origin AUTH-42/01-database-schema
```

---

## Advanced Techniques

### Technique 1: Cherry-Picking Between Layers

```bash
# Made change in layer 03, should be in layer 01
# Cherry-pick it back

# Identify the change
jj show AUTH-42/03-api-endpoints
# See: migrations/add_index.sql (oops, wrong layer!)

# Create change in correct layer
jj new -B AUTH-42/01-database-schema --no-edit
# Manually copy migrations/add_index.sql
jj describe -m "Add index on user.email"
jj squash --into AUTH-42/01-database-schema

# Remove from layer 03
jj new -B AUTH-42/03-api-endpoints --no-edit
# Revert migrations/add_index.sql
jj squash --into AUTH-42/03-api-endpoints
```

### Technique 2: Parallel Stacks for Experimental Work

```bash
# Main stack
AUTH-42/01-database-schema
AUTH-42/02-core-logic
AUTH-42/dev

# Experimental parallel stack
AUTH-42-exp/01-alternative-approach
AUTH-42-exp/02-test-performance
AUTH-42-exp/dev

# If experiment succeeds, merge into main stack
# If experiment fails, delete experimental bookmarks
```

### Technique 3: Bookmark Aliases for Navigation

```bash
# Create short aliases in your shell config
alias jj-stack='jj log -r "AUTH-42/root::AUTH-42/dev"'
alias jj-layer1='jj new -B AUTH-42/01-database-schema --no-edit'
alias jj-layer2='jj new -B AUTH-42/02-core-logic --no-edit'

# Use them
jj-stack      # View entire stack
jj-layer1     # Jump to layer 1 for work
```

### Technique 4: Automated Stack Verification

```bash
#!/bin/bash
# verify-stack.sh

STACK_ID="AUTH-42"

echo "Verifying stack: $STACK_ID"

# Check linear history
if jj log -r "${STACK_ID}/root::${STACK_ID}/dev" --no-graph | grep -q "merge"; then
    echo "ERROR: Stack has merges (should be linear)"
    exit 1
fi

# Check all numbered bookmarks exist
for i in 01 02 03; do
    if ! jj bookmark list | grep -q "${STACK_ID}/${i}-"; then
        echo "WARNING: Missing bookmark ${STACK_ID}/${i}-*"
    fi
done

echo "Stack verification complete"
```

### Technique 5: Squashing Multiple Commits

```bash
# Made several commits in dev, want to squash into layer 02

# Option A: Interactive squash
jj new -B AUTH-42/02-core-logic --no-edit
jj squash --from <commit1> --from <commit2> --from <commit3>

# Option B: Squash entire working set
jj new -B AUTH-42/02-core-logic --no-edit
# Make multiple commits
jj squash --into AUTH-42/02-core-logic  # Squashes everything
```

### Technique 6: Splitting Commits Across Layers

```bash
# One commit touches files for both layer 01 and layer 02
# Split it using jj split

jj edit <commit-with-mixed-changes>
jj split

# jj opens interactive split
# Select which files go to first commit (layer 01 files)
# Remaining files go to second commit (layer 02 files)

# Now move each piece to appropriate layer
jj squash -r <first-commit> --into AUTH-42/01-database-schema
jj squash -r <second-commit> --into AUTH-42/02-core-logic
```

### Technique 7: Stack Templates

```bash
#!/bin/bash
# create-stack.sh <TASK-ID> <layer1-name> <layer2-name> ...

TASK_ID=$1
shift

# Create root
jj new main -m "Start $TASK_ID stack"
jj bookmark create ${TASK_ID}/root

# Create layers
PREV="${TASK_ID}/root"
INDEX=1
for LAYER_NAME in "$@"; do
    LAYER_NUM=$(printf "%02d" $INDEX)
    jj new $PREV -m "$LAYER_NAME"
    jj bookmark create ${TASK_ID}/${LAYER_NUM}-${LAYER_NAME}
    PREV="${TASK_ID}/${LAYER_NUM}-${LAYER_NAME}"
    INDEX=$((INDEX + 1))
done

# Create dev
jj new $PREV -m "WIP"
jj bookmark create ${TASK_ID}/dev

echo "Stack created: ${TASK_ID}/root -> ${TASK_ID}/dev"
jj log -r "${TASK_ID}/root::${TASK_ID}/dev"
```

Usage:
```bash
./create-stack.sh AUTH-42 database-schema core-logic api-endpoints tests
```

---

## Complete Example Workflow

Let's walk through a complete feature from start to finish.

### Scenario: Add Authentication Feature

**Goal:** Add user authentication with login/register

**Plan:**
1. Layer 01: Database schema (users, sessions tables)
2. Layer 02: Core logic (password hashing, session management)
3. Layer 03: API endpoints (login, register, logout)
4. Layer 04: Tests

### Step 1: Initialize Stack

```bash
# Start from main
jj git fetch
jj new main -m "Start AUTH-42 authentication stack"
jj bookmark create AUTH-42/root

# Create layer structure
jj new AUTH-42/root -m "Database schema"
jj bookmark create AUTH-42/01-database-schema

jj new AUTH-42/01-database-schema -m "Core authentication logic"
jj bookmark create AUTH-42/02-core-logic

jj new AUTH-42/02-core-logic -m "API endpoints"
jj bookmark create AUTH-42/03-api-endpoints

jj new AUTH-42/03-api-endpoints -m "Tests"
jj bookmark create AUTH-42/04-tests

jj new AUTH-42/04-tests -m "WIP"
jj bookmark create AUTH-42/dev

# Verify
jj log -r 'AUTH-42/root::AUTH-42/dev'
```

### Step 2: Implement Layer 01 (Database)

```bash
# Work on database schema
jj new -B AUTH-42/01-database-schema --no-edit

# Create migration
cat > migrations/001_create_users.sql << 'EOF'
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_users_email ON users(email);
EOF

# Create model
cat > models/user.py << 'EOF'
from sqlalchemy import Column, Integer, String, DateTime
from database import Base

class User(Base):
    __tablename__ = 'users'
    id = Column(Integer, primary_key=True)
    email = Column(String(255), unique=True, nullable=False)
    password_hash = Column(String(255), nullable=False)
    created_at = Column(DateTime)
EOF

# Commit and squash
jj describe -m "Add users table and User model"
jj squash --into AUTH-42/01-database-schema

# Verify
jj show AUTH-42/01-database-schema
```

### Step 3: Implement Layer 02 (Core Logic)

```bash
# Work on core logic
jj new -B AUTH-42/02-core-logic --no-edit

# Create auth service
cat > services/auth_service.py << 'EOF'
import bcrypt
from models.user import User

def hash_password(password: str) -> str:
    return bcrypt.hashpw(password.encode(), bcrypt.gensalt()).decode()

def verify_password(password: str, password_hash: str) -> bool:
    return bcrypt.checkpw(password.encode(), password_hash.encode())

def create_user(email: str, password: str) -> User:
    password_hash = hash_password(password)
    user = User(email=email, password_hash=password_hash)
    return user
EOF

jj describe -m "Implement password hashing and user creation"
jj squash --into AUTH-42/02-core-logic

# Verify
jj show AUTH-42/02-core-logic
```

### Step 4: Implement Layer 03 (API)

```bash
# Work on API endpoints
jj new -B AUTH-42/03-api-endpoints --no-edit

# Create routes
cat > api/auth_routes.py << 'EOF'
from fastapi import APIRouter, HTTPException
from services.auth_service import create_user, verify_password
from models.user import User

router = APIRouter()

@router.post("/register")
def register(email: str, password: str):
    user = create_user(email, password)
    return {"message": "User created", "email": user.email}

@router.post("/login")
def login(email: str, password: str):
    # Implementation here
    pass
EOF

jj describe -m "Add register and login endpoints"
jj squash --into AUTH-42/03-api-endpoints
```

### Step 5: Add Tests (Layer 04)

```bash
# Work on tests
jj new -B AUTH-42/04-tests --no-edit

# Create test file
cat > tests/test_auth.py << 'EOF'
import pytest
from services.auth_service import hash_password, verify_password

def test_password_hashing():
    password = "secret123"
    hashed = hash_password(password)
    assert verify_password(password, hashed)
    assert not verify_password("wrong", hashed)
EOF

jj describe -m "Add authentication tests"
jj squash --into AUTH-42/04-tests
```

### Step 6: Publish PRs

```bash
# Push all layers
jj git push --bookmark AUTH-42/01-database-schema
jj git push --bookmark AUTH-42/02-core-logic
jj git push --bookmark AUTH-42/03-api-endpoints
jj git push --bookmark AUTH-42/04-tests
```

**On GitHub:**

**PR #1234: [AUTH-42][1/4] Add database schema**
- Base: `main`
- Files: `migrations/001_create_users.sql`, `models/user.py`

**PR #1235: [AUTH-42][2/4] Implement core authentication logic**
- Base: `AUTH-42/01-database-schema`
- Depends on: #1234
- Files: `services/auth_service.py`

**PR #1236: [AUTH-42][3/4] Add API endpoints**
- Base: `AUTH-42/02-core-logic`
- Depends on: #1235
- Files: `api/auth_routes.py`

**PR #1237: [AUTH-42][4/4] Add authentication tests**
- Base: `AUTH-42/03-api-endpoints`
- Depends on: #1236
- Files: `tests/test_auth.py`

### Step 7: Handle Review Feedback

**Reviewer comment on PR #1234:** "Add index on created_at"

```bash
# Make change in layer 01
jj new -B AUTH-42/01-database-schema --no-edit

# Update migration
cat >> migrations/001_create_users.sql << 'EOF'
CREATE INDEX idx_users_created_at ON users(created_at);
EOF

jj describe -m "Add index on users.created_at"
jj squash --into AUTH-42/01-database-schema

# Push update
jj git push --bookmark AUTH-42/01-database-schema --force
```

**Layers 02, 03, 04 automatically rebased!**

### Step 8: Merge PRs

```bash
# PR #1234 approved and merged to main
jj git fetch

# Update root
jj bookmark set AUTH-42/root -r main

# Delete merged bookmark
jj bookmark delete AUTH-42/01-database-schema

# Update layer 02 to build on main
jj rebase -s AUTH-42/02-core-logic -d main

# Continue with remaining PRs...
```

---

## Summary

This bookmark-based stacked PR workflow provides:

✅ **Clear structure** - Numbered layers show dependencies
✅ **Flexible development** - Add work anywhere with `jj new -B` + `squash`
✅ **Safe iterations** - Operation log protects against mistakes
✅ **Clean history** - Linear chain from root to dev
✅ **Easy collaboration** - Handle external changes gracefully
✅ **Independent review** - Each layer reviewable separately

**Key Commands:**
```bash
jj new -B BOOKMARK --no-edit    # Start work at bookmark
jj squash --into BOOKMARK        # Squash work into bookmark
jj git push --bookmark BOOKMARK  # Publish layer
jj undo                       # Undo mistakes
```

**Remember:**
- Work upstream (early layers) when possible
- Keep layers focused and independent
- Use descriptive bookmark names
- Document dependencies in PRs
- Communicate force pushes
- Regular stack hygiene


CRITICAL:
    Ask the user what task mentioned in this document should be performed.
    If there's no explicit intent for stacked PRs, do not create stacked PRs without user consent.
