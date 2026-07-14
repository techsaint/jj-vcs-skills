# Converting an Existing Branch to Stacked PRs

## Overview

You've built a feature on a single branch with many commits, and now you need to break it into reviewable stacked PRs. This guide shows how to use jj's powerful history editing to reorganize commits into logical layers, even when the original commits don't align with the desired grouping.

**What You'll Learn:**
- Analyzing an existing branch to identify logical layers
- Creating a stack structure from scratch
- Moving changes between commits to create clean layers
- Splitting monolithic commits across multiple layers
- Merging related commits into single layers
- Handling dependencies and conflicts during reorganization
- Validating the reorganized stack before publishing

**Key jj Features Used:**
- `jj split` - Split commits into smaller pieces
- `jj squash` - Combine commits (including path-aware `--from`/`--into` moves)
- `jj restore` - Copy path content from one revision into another
- `jj new` and `jj edit` - Position and modify commits
- `jj rebase` - Reorganize commit relationships
- `jj bookmark` - Mark and organize layers

## Table of Contents

1. [When to Convert vs Start Fresh](#when-to-convert-vs-start-fresh)
2. [Analysis Phase](#analysis-phase)
3. [Planning the Stack Structure](#planning-the-stack-structure)
4. [Conversion Strategies](#conversion-strategies)
5. [Strategy 1: Bottom-Up Extraction](#strategy-1-bottom-up-extraction)
6. [Strategy 2: Top-Down Reorganization](#strategy-2-top-down-reorganization)
7. [Strategy 3: Surgical File Movement](#strategy-3-surgical-file-movement)
8. [Handling Common Scenarios](#handling-common-scenarios)
9. [Validation and Testing](#validation-and-testing)
10. [Complete Examples](#complete-examples)
11. [Troubleshooting](#troubleshooting)
12. [Best Practices](#best-practices)

---

## When to Convert vs Start Fresh

### Convert When:

✅ **Significant work already done** - 10+ commits, days/weeks of work
✅ **Working code** - Tests pass, feature works
✅ **Logical separation possible** - Can identify distinct layers
✅ **History has value** - Commit messages/context worth preserving
✅ **Time constrained** - Need to get to review quickly

### Start Fresh When:

❌ **Experimental/messy work** - "WIP", "fix", "oops" commits everywhere
❌ **Major rework needed** - Feature needs redesign
❌ **Small amount of work** - 3-5 commits, easy to redo
❌ **No clear layers** - Everything intertwined
❌ **Learning opportunity** - Practicing good workflow

**Rule of thumb:** If you can identify 3+ logical layers in the existing work, convert it. Otherwise, consider starting fresh with the stacked PR workflow.

---

## Analysis Phase

### Step 1: Review Existing Commits

```bash
# See all commits in your branch
jj log -r 'main..@'

# More detailed view with files changed
jj log -r 'main..@' --summary

# See full diff of entire branch
jj diff -r main..@
```

**What to look for:**
- How many commits? (Too many → combine, too few → split)
- What files are touched repeatedly? (Indicates layer boundaries)
- Are commits logical? (Or random groupings?)
- What's the dependency order? (Database → Logic → API → Tests)

### Step 2: Identify File Groupings

```bash
# List all files changed in the branch
jj diff -r main..@ --stat

# See which commits touched specific files
jj log -r 'main..@' --summary | grep "filename"
```

**Create a file inventory:**

```
Database/Schema:
  - migrations/001_add_users.sql
  - migrations/002_add_sessions.sql
  - models/user.py
  - models/session.py

Core Logic:
  - services/auth_service.py
  - services/session_manager.py
  - utils/password.py

API Layer:
  - api/auth_routes.py
  - api/schemas/auth.py
  - api/middleware/session.py

Tests:
  - tests/test_auth_service.py
  - tests/test_session_manager.py
  - tests/api/test_auth_routes.py

Config/Infrastructure:
  - config/security.py
  - requirements.txt
```

### Step 3: Map Commits to Desired Layers

Create a mapping table:

```
CURRENT COMMITS                    DESIRED LAYER
-----------------                  --------------
commit1: "Add User model"       →  Layer 01: Database Schema
commit2: "Add Session model"    →  Layer 01: Database Schema
commit3: "Add password utils"   →  Layer 02: Core Logic
commit4: "Add auth endpoints"   →  Layer 03: API (+ some to Layer 02)
commit5: "Add session mgmt"     →  Layer 02: Core Logic
commit6: "Add tests"            →  Layer 04: Tests
commit7: "Update requirements"  →  Layer 01: Database Schema
commit8: "Fix auth bug"         →  Layer 02: Core Logic
```

**Red flags to note:**
- Commits that span multiple layers (need to split)
- Related functionality in separate commits (need to combine)
- Wrong order (Layer 03 work before Layer 01)
- Missing pieces (gaps in functionality)

### Step 4: Identify Dependencies

Draw dependency graph:

```
User Model ──┬──> Auth Service ──> Auth API
             │                        ↓
Session Model┴──> Session Manager ──> Tests
                         ↓
                  Password Utils
```

**This reveals the natural layer structure:**
```
Layer 01: Models (User, Session)
Layer 02: Core Services (Auth, Session, Password Utils)
Layer 03: API (Routes, Schemas)
Layer 04: Tests
```

---

## Planning the Stack Structure

### Define Your Layers

Based on analysis, create a plan:

```
TASK-123/root (= main)
    ↓
TASK-123/01-database-models
    - migrations/001_add_users.sql
    - migrations/002_add_sessions.sql
    - models/user.py
    - models/session.py
    - requirements.txt (sqlalchemy)
    ↓
TASK-123/02-core-services
    - services/auth_service.py
    - services/session_manager.py
    - utils/password.py
    - requirements.txt (bcrypt)
    ↓
TASK-123/03-api-endpoints
    - api/auth_routes.py
    - api/schemas/auth.py
    - api/middleware/session.py
    - requirements.txt (fastapi)
    ↓
TASK-123/04-tests
    - tests/test_auth_service.py
    - tests/test_session_manager.py
    - tests/api/test_auth_routes.py
    - requirements.txt (pytest)
    ↓
TASK-123/dev
```

### Document the Transformation Plan

Create a checklist:

```markdown
## Conversion Checklist

### Preparation
- [x] Analyzed existing commits
- [x] Identified file groupings
- [x] Mapped commits to layers
- [x] Identified dependencies
- [ ] Backed up current state (jj op log)

### Layer 01: Database Models
- [ ] Create bookmark TASK-123/01-database-models
- [ ] Extract migrations from commits 1, 2, 7
- [ ] Extract model files from commits 1, 2
- [ ] Extract SQLAlchemy from requirements.txt

### Layer 02: Core Services
- [ ] Create bookmark TASK-123/02-core-services
- [ ] Extract auth_service.py from commits 3, 8
- [ ] Extract session_manager.py from commit 5
- [ ] Extract password utils from commit 3
- [ ] Extract bcrypt from requirements.txt

### Layer 03: API Endpoints
- [ ] Create bookmark TASK-123/03-api-endpoints
- [ ] Extract API routes from commit 4
- [ ] Extract schemas from commit 4
- [ ] Extract middleware from commit 5
- [ ] Extract FastAPI from requirements.txt

### Layer 04: Tests
- [ ] Create bookmark TASK-123/04-tests
- [ ] Extract all test files from commit 6
- [ ] Extract pytest from requirements.txt

### Validation
- [ ] Verify linear history
- [ ] Run tests at each layer
- [ ] Check diff against original branch
- [ ] Verify no files lost
```

---

## Conversion Strategies

There are three main strategies for converting a branch to stacked PRs:

### Strategy Comparison

| Strategy | Best For | Difficulty | Preserves History |
|----------|----------|------------|-------------------|
| Bottom-Up Extraction | Clean commits, clear layers | Medium | Partially |
| Top-Down Reorganization | Messy commits, needs rewrite | Hard | No |
| Surgical File Movement | Mixed situation | Medium-Hard | Partially |

### Choosing a Strategy

**Use Bottom-Up when:**
- Commits are mostly logical
- Each commit touches distinct areas
- You want to preserve commit messages
- Work is mostly in correct order

**Use Top-Down when:**
- Commits are a mess
- Heavy reorganization needed
- You don't care about preserving history
- Fresh commit messages would be better

**Use Surgical File Movement when:**
- Some commits are good, some need work
- Files are scattered across commits
- You want precise control
- Intermediate between the other two

---

## Strategy 1: Bottom-Up Extraction

**Philosophy:** Start with existing commits, extract and reorganize them into layers.

### Step 1: Create Empty Stack Structure

```bash
# Create root
jj bookmark create TASK-123/root -r main

# Create empty layer bookmarks
jj new main -m "Database models"
jj bookmark create TASK-123/01-database-models

jj new TASK-123/01-database-models -m "Core services"
jj bookmark create TASK-123/02-core-services

jj new TASK-123/02-core-services -m "API endpoints"
jj bookmark create TASK-123/03-api-endpoints

jj new TASK-123/03-api-endpoints -m "Tests"
jj bookmark create TASK-123/04-tests

jj new TASK-123/04-tests -m "WIP"
jj bookmark create TASK-123/dev

# Verify structure
jj log -r 'TASK-123/root::TASK-123/dev'
```

### Step 2: Extract Layer 01 Content

```bash
# View original commits
jj log -r 'main..@' --summary

# Find commits with database/model changes
# Example: commit1 (abc123) and commit2 (def456)

# Method A: Cherry-pick specific files from commits
jj new -B TASK-123/01-database-models --no-edit

# Restore files from specific commits
jj restore --from abc123 migrations/001_add_users.sql
jj restore --from abc123 models/user.py
jj restore --from def456 migrations/002_add_sessions.sql
jj restore --from def456 models/session.py

# Update requirements.txt manually (just SQLAlchemy part)
# Edit requirements.txt

jj describe -m "Add User and Session database models

Migrations:
- 001_add_users.sql: Create users table with email, password_hash
- 002_add_sessions.sql: Create sessions table with token, expiry

Models:
- User model with authentication fields
- Session model with token management"

jj squash --into TASK-123/01-database-models
```

**Alternative Method B: Use jj squash**

```bash
# Move specific changes from old commits to layer
jj new -B TASK-123/01-database-models --no-edit

# Move changes from old commits
jj squash --from abc123 --into @ migrations/ models/user.py
jj squash --from def456 --into @ migrations/002_add_sessions.sql models/session.py

jj describe -m "Add User and Session database models"
jj squash --into TASK-123/01-database-models
```

### Step 3: Extract Layer 02 Content

```bash
# Extract core services
jj new -B TASK-123/02-core-services --no-edit

# Find commits with service code
# commit3 (ghi789), commit5 (mno012), commit8 (stu345)

jj restore --from ghi789 services/auth_service.py utils/password.py
jj restore --from mno012 services/session_manager.py
jj restore --from stu345 services/auth_service.py  # Bug fix

# Edit requirements.txt for bcrypt
# Edit requirements.txt

jj describe -m "Implement authentication and session services

Services:
- auth_service.py: User authentication with password hashing
- session_manager.py: Session creation and validation
- password.py: Bcrypt password utilities

Includes fix for authentication edge case from commit stu345"

jj squash --into TASK-123/02-core-services
```

### Step 4: Extract Layer 03 Content

```bash
# Extract API layer
jj new -B TASK-123/03-api-endpoints --no-edit

# commit4 (jkl345) has API work
jj restore --from jkl345 api/

# But it might also have some service code that belongs in layer 02
# Check what was restored
jj diff

# If there's service code, remove it
git restore --staged services/  # Undo service changes
jj restore services/  # Remove from working copy

# Edit requirements.txt for FastAPI
# Edit requirements.txt

jj describe -m "Add authentication API endpoints

Endpoints:
- POST /auth/register: User registration
- POST /auth/login: User login with session creation
- POST /auth/logout: Session invalidation
- GET /auth/session: Session validation

Includes request/response schemas and session middleware"

jj squash --into TASK-123/03-api-endpoints
```

### Step 5: Extract Layer 04 Content

```bash
# Extract tests
jj new -B TASK-123/04-tests --no-edit

# commit6 (pqr678) has tests
jj restore --from pqr678 tests/

# Edit requirements.txt for pytest
# Edit requirements.txt

jj describe -m "Add comprehensive test suite

Tests:
- test_auth_service.py: Unit tests for authentication logic
- test_session_manager.py: Unit tests for session management
- test_auth_routes.py: Integration tests for API endpoints

All tests passing with >90% coverage"

jj squash --into TASK-123/04-tests
```

### Step 6: Verify and Clean Up

```bash
# View your new stack
jj log -r 'TASK-123/root::TASK-123/dev'

# Compare with original branch
jj diff -r main..TASK-123/04-tests --stat
jj diff -r main..@original-branch --stat

# Should be identical (or explain differences)

# Run tests at each layer
jj new TASK-123/01-database-models
# Run: pytest tests/test_models.py (if any)

jj new TASK-123/02-core-services
# Run: pytest tests/test_auth_service.py tests/test_session_manager.py

jj new TASK-123/03-api-endpoints
# Run: pytest tests/api/ (might fail - tests in layer 04)

jj new TASK-123/04-tests
# Run: pytest (all should pass)
```

---

## Strategy 2: Top-Down Reorganization

**Philosophy:** Create clean layers from scratch, copying code from the messy branch as needed.

### Step 1: Create Working Copy of Original Branch

```bash
# Bookmark your current messy work
jj bookmark create TASK-123/original

# Create root for new stack
jj new main -m "Start clean stack"
jj bookmark create TASK-123/root
```

### Step 2: Build Layer 01 From Scratch

```bash
# Create clean layer 01
jj new TASK-123/root -m "Database models"
jj bookmark create TASK-123/01-database-models

# Start working on it
jj new -B TASK-123/01-database-models --no-edit

# Manually copy/create files from original branch
# View original for reference
jj show TASK-123/original --summary

# Copy database migrations
cp path/to/original/migrations/001_add_users.sql migrations/
cp path/to/original/migrations/002_add_sessions.sql migrations/

# Or use jj restore selectively
jj restore --from TASK-123/original migrations/ models/

# Clean up what you don't need
rm models/unrelated_stuff.py

# Create a clean, well-organized commit
jj describe -m "Add database schema for authentication

Database Tables:
- users: id, email, password_hash, created_at
- sessions: id, user_id, token, expires_at

SQLAlchemy Models:
- User: Handles user data and relationships
- Session: Manages session lifecycle

Migration files create indexes for common queries."

jj squash --into TASK-123/01-database-models
```

### Step 3: Build Layer 02 From Scratch

```bash
# Create clean layer 02
jj new TASK-123/01-database-models -m "Core services"
jj bookmark create TASK-123/02-core-services

jj new -B TASK-123/02-core-services --no-edit

# Selectively copy service files
jj restore --from TASK-123/original services/ utils/

# But maybe auth_service.py needs refactoring
# Edit services/auth_service.py
# - Clean up the code
# - Fix the bug that was in commit8
# - Organize better

# Write a clean commit message explaining the clean version
jj describe -m "Implement authentication and session management

Core Services:
- AuthService: User registration, login, password verification
  - Uses bcrypt for password hashing (work factor: 12)
  - Returns JWT tokens for authenticated sessions

- SessionManager: Session lifecycle management
  - Creates sessions with secure random tokens
  - Handles session expiration (24hr default)
  - Validates and refreshes sessions

- PasswordUtils: Password hashing utilities
  - Bcrypt wrapper with sensible defaults
  - Password strength validation

Improvements over original implementation:
- Fixed authentication edge case with empty passwords
- Added session token rotation
- Improved error messages"

jj squash --into TASK-123/02-core-services
```

### Step 4: Build Remaining Layers

Continue the same pattern for layers 03 and 04:

```bash
# Layer 03: API
jj new TASK-123/02-core-services -m "API endpoints"
jj bookmark create TASK-123/03-api-endpoints

jj new -B TASK-123/03-api-endpoints --no-edit
jj restore --from TASK-123/original api/
# Refactor/clean as needed
jj describe -m "Add RESTful authentication API"
jj squash --into TASK-123/03-api-endpoints

# Layer 04: Tests
jj new TASK-123/03-api-endpoints -m "Test suite"
jj bookmark create TASK-123/04-tests

jj new -B TASK-123/04-tests --no-edit
jj restore --from TASK-123/original tests/
# Update tests to match refactored code
jj describe -m "Add comprehensive test coverage"
jj squash --into TASK-123/04-tests
```

### Step 5: Validate and Delete Original

```bash
# Ensure everything works
jj new TASK-123/04-tests
pytest  # All tests should pass

# Compare final result with original
jj diff -r TASK-123/original..TASK-123/04-tests

# If satisfied, delete original bookmark
jj bookmark delete TASK-123/original
```

**Advantages of Top-Down:**
- Clean commit messages
- Can refactor as you go
- No history baggage
- Clear separation of concerns

**Disadvantages:**
- Loses original commit history
- More manual work
- Need to understand all code

---

## Strategy 3: Surgical File Movement

**Philosophy:** Precisely move individual file changes between commits to reorganize.

### Step 1: Set Up Parallel Stack

```bash
# Keep original branch
jj bookmark create TASK-123/original

# Create empty stack
jj new main -m "Database models"
jj bookmark create TASK-123/01-database-models

jj new TASK-123/01-database-models -m "Core services"
jj bookmark create TASK-123/02-core-services

jj new TASK-123/02-core-services -m "API endpoints"
jj bookmark create TASK-123/03-api-endpoints

jj new TASK-123/03-api-endpoints -m "Tests"
jj bookmark create TASK-123/04-tests

jj new TASK-123/04-tests -m "WIP"
jj bookmark create TASK-123/dev
```

### Step 2: Use path-aware `jj squash --from/--into` for Precision

On jj 0.40+, move changes between commits with path-aware squash (there is no
`jj move` subcommand):

```bash
# Syntax: jj squash --from SOURCE --into DEST [files...]

# Example: Move migrations from original commit to layer 01

# First, identify the commit with migrations
jj log -r 'main..TASK-123/original' --summary
# Found: commit abc123 has migrations/001_add_users.sql

# Move just the migrations
jj squash --from abc123 --into TASK-123/01-database-models migrations/001_add_users.sql

# Move models from multiple commits
jj squash --from abc123 --into TASK-123/01-database-models models/user.py
jj squash --from def456 --into TASK-123/01-database-models models/session.py
```

### Step 3: Move Files to Appropriate Layers

```bash
# Systematically move each file type

# Database layer (01)
jj squash --from <various-commits> --into TASK-123/01-database-models migrations/ models/

# Core services layer (02)
jj squash --from <various-commits> --into TASK-123/02-core-services services/ utils/

# API layer (03)
jj squash --from <various-commits> --into TASK-123/03-api-endpoints api/

# Tests layer (04)
jj squash --from <various-commits> --into TASK-123/04-tests tests/
```

### Step 4: Handle Split Files

Sometimes one file has changes that belong in multiple layers:

```bash
# Example: requirements.txt has dependencies for all layers

# View what's in requirements.txt across commits
jj show TASK-123/original requirements.txt

# Content:
# sqlalchemy==2.0.0    # Layer 01
# bcrypt==4.0.0        # Layer 02
# fastapi==0.100.0     # Layer 03
# pytest==7.4.0        # Layer 04

# Split it manually across layers

# Layer 01: Just SQLAlchemy
jj new -B TASK-123/01-database-models --no-edit
cat > requirements.txt << EOF
sqlalchemy==2.0.0
EOF
jj squash --into TASK-123/01-database-models

# Layer 02: Add bcrypt
jj new -B TASK-123/02-core-services --no-edit
cat >> requirements.txt << EOF
bcrypt==4.0.0
EOF
jj squash --into TASK-123/02-core-services

# Layer 03: Add FastAPI
jj new -B TASK-123/03-api-endpoints --no-edit
cat >> requirements.txt << EOF
fastapi==0.100.0
EOF
jj squash --into TASK-123/03-api-endpoints

# Layer 04: Add pytest
jj new -B TASK-123/04-tests --no-edit
cat >> requirements.txt << EOF
pytest==7.4.0
EOF
jj squash --into TASK-123/04-tests
```

### Step 5: Verify No Changes Lost

```bash
# Check that all changes made it to the stack
jj diff -r main..TASK-123/04-tests --stat > new-stack-files.txt
jj diff -r main..TASK-123/original --stat > original-files.txt
diff new-stack-files.txt original-files.txt

# Should show no differences (or only intentional ones)
```

**Advantages of Surgical Movement:**
- Precise control over what goes where
- Can preserve some commit structure
- Good for mixed situations
- Easy to verify

**Disadvantages:**
- Can be tedious with many files
- Requires understanding jj squash well
- Manual description updates needed

---

## Handling Common Scenarios

### Scenario 1: Monolithic Commit Spanning Multiple Layers

**Problem:** One commit changes database, services, and API.

```bash
# Original commit abc123 has:
# - migrations/001_add_users.sql
# - models/user.py
# - services/auth_service.py
# - api/auth_routes.py

# Solution: Split the commit, move parts to appropriate layers

# Method A: Use jj split
jj edit abc123
jj split

# jj will ask which files go in first commit
# Select: migrations/ and models/
# Rest goes to second commit

# Now move each piece
jj squash --from <first-split> --into TASK-123/01-database-models migrations/ models/
jj squash --from <second-split> --into TASK-123/02-core-services services/
jj squash --from <second-split> --into TASK-123/03-api-endpoints api/
```

**Method B: Manual extraction**

```bash
# Don't split, just extract what you need

# Layer 01: Extract database parts
jj new -B TASK-123/01-database-models --no-edit
jj restore --from abc123 migrations/ models/
jj squash --into TASK-123/01-database-models

# Layer 02: Extract service parts
jj new -B TASK-123/02-core-services --no-edit
jj restore --from abc123 services/
jj squash --into TASK-123/02-core-services

# Layer 03: Extract API parts
jj new -B TASK-123/03-api-endpoints --no-edit
jj restore --from abc123 api/
jj squash --into TASK-123/03-api-endpoints
```

### Scenario 2: Changes in Wrong Order

**Problem:** API endpoint created before the service it depends on.

```bash
# Original history:
# commit1: Add API endpoint (depends on AuthService)
# commit2: Add AuthService

# Need to reverse order for stack

# Solution: Extract in logical order, not historical order

# Layer 02 (Core Services) - extract from commit2
jj new -B TASK-123/02-core-services --no-edit
jj restore --from commit2 services/auth_service.py
jj squash --into TASK-123/02-core-services

# Layer 03 (API) - extract from commit1
jj new -B TASK-123/03-api-endpoints --no-edit
jj restore --from commit1 api/auth_routes.py
jj squash --into TASK-123/03-api-endpoints

# Result: Clean dependency order, even though original was reversed
```

### Scenario 3: Bug Fix That Needs to Be Integrated

**Problem:** Bug fix commit in middle of branch.

```bash
# Original:
# commit1: Add AuthService
# commit2: Add API
# commit3: Fix AuthService bug  ← Should be merged into commit1
# commit4: Add tests

# Solution: Combine the original and fix into one layer

# Layer 02: Include both original and fix
jj new -B TASK-123/02-core-services --no-edit
jj restore --from commit1 services/auth_service.py
jj restore --from commit3 services/auth_service.py  # Overwrites with fixed version
jj describe -m "Implement AuthService

Includes fix for edge case with empty passwords (originally commit3)"
jj squash --into TASK-123/02-core-services
```

### Scenario 4: Refactoring Needed During Conversion

**Problem:** Code structure doesn't match desired layer structure.

```bash
# Original: auth_service.py has both core logic and API helpers
# Desired: Core logic in Layer 02, API helpers in Layer 03

# Solution: Split the file during conversion

# Layer 02: Extract core logic only
jj new -B TASK-123/02-core-services --no-edit
jj restore --from original-commit services/auth_service.py

# Edit the file to remove API helper functions
# Edit services/auth_service.py
# Remove: format_auth_response(), validate_api_token(), etc.

jj describe -m "Implement core authentication service"
jj squash --into TASK-123/02-core-services

# Layer 03: Create new file for API helpers
jj new -B TASK-123/03-api-endpoints --no-edit
jj restore --from original-commit services/auth_service.py

# Extract just the API helpers to new location
# Edit api/auth_helpers.py (create new file)
# Copy format_auth_response(), validate_api_token(), etc.
# Update imports

jj describe -m "Add API authentication helpers"
jj squash --into TASK-123/03-api-endpoints
```

### Scenario 5: Incomplete Features Across Commits

**Problem:** Feature partially implemented across multiple commits.

```bash
# Original:
# commit1: Start user registration (backend only, incomplete)
# commit2: Add login (complete)
# commit3: Finish user registration (add validation)

# Desired: Complete registration in one layer

# Solution: Combine related commits

# Layer: Complete registration feature
jj new -B TASK-123/02-core-services --no-edit
jj restore --from commit1 services/registration.py
jj restore --from commit3 services/registration.py  # Adds validation

jj describe -m "Implement complete user registration

Includes:
- Email/password registration
- Input validation (from commit3)
- Password strength requirements
- Email uniqueness checks"

jj squash --into TASK-123/02-core-services
```

### Scenario 6: Experimental Code to Remove

**Problem:** Branch has experimental code that shouldn't be in PRs.

```bash
# Original:
# commit1: Add AuthService
# commit2: Add experimental OAuth (not ready)
# commit3: Add tests

# Solution: Simply don't extract the experimental commit

# Layer 02: Skip commit2 entirely
jj new -B TASK-123/02-core-services --no-edit
jj restore --from commit1 services/auth_service.py
# Notice: NOT restoring from commit2
jj squash --into TASK-123/02-core-services

# Layer 03: Continue with tests
jj new -B TASK-123/04-tests --no-edit
jj restore --from commit3 tests/
jj squash --into TASK-123/04-tests

# Experimental code stays in original branch for later
```

---

## Validation and Testing

### Pre-Validation Checklist

Before declaring conversion complete:

```bash
# 1. Check linear history
jj log -r 'TASK-123/root::TASK-123/dev' --graph
# Should be straight line, no merges

# 2. Verify all bookmarks exist
jj bookmark list | grep TASK-123
# Should see: root, 01, 02, 03, 04, dev

# 3. Check file coverage
jj diff -r main..TASK-123/04-tests --stat > converted-files.txt
jj diff -r main..TASK-123/original --stat > original-files.txt
diff converted-files.txt original-files.txt
# Should be identical (or explain differences)
```

### Test Each Layer Independently

```bash
# Layer 01: Database models
jj new TASK-123/01-database-models

# Run database migrations
alembic upgrade head

# Test models can be imported
python -c "from models.user import User; from models.session import Session; print('OK')"

# Layer 02: Core services
jj new TASK-123/02-core-services

# Run service tests (if any exist at this layer)
pytest tests/test_auth_service.py tests/test_session_manager.py -v

# Test services can be imported and work
python -c "from services.auth_service import AuthService; AuthService().hash_password('test')"

# Layer 03: API endpoints
jj new TASK-123/03-api-endpoints

# Start API server (if possible)
uvicorn api.main:app --reload &
sleep 2

# Test endpoints respond
curl http://localhost:8000/auth/health

# Kill server
pkill uvicorn

# Layer 04: Tests
jj new TASK-123/04-tests

# Run full test suite
pytest -v

# Check coverage
pytest --cov=. --cov-report=term-missing
```

### Compare Behavior with Original

```bash
# Run tests on original branch
jj new TASK-123/original
pytest -v > original-test-results.txt

# Run tests on converted stack
jj new TASK-123/04-tests
pytest -v > converted-test-results.txt

# Compare
diff original-test-results.txt converted-test-results.txt
# Should be identical (all same tests passing)
```

### Validate No Lost Changes

```bash
# Create diff of original vs converted
jj diff -r TASK-123/original..TASK-123/04-tests > conversion-diff.patch

# Should be empty (or only intentional changes)
wc -l conversion-diff.patch
# If non-zero, review carefully:
cat conversion-diff.patch
```

### Performance Sanity Check

```bash
# If you refactored, ensure performance is similar

# Original branch
jj new TASK-123/original
time pytest tests/test_auth_service.py
# Note: 2.34s

# Converted stack
jj new TASK-123/04-tests
time pytest tests/test_auth_service.py
# Should be similar: 2.41s (within 20%)
```

---

## Complete Examples

### Example 1: Simple Linear Conversion

**Scenario:** 5 commits, already mostly organized.

```bash
# Original commits:
# abc123: Add User model
# def456: Add AuthService
# ghi789: Add API routes
# jkl012: Add tests
# mno345: Fix typo in AuthService

# Analysis: Mostly good, just need to merge fix into service

# Step 1: Create stack
jj bookmark create TASK-123/root -r main

jj new main -m "Database models"
jj bookmark create TASK-123/01-database-models

jj new TASK-123/01-database-models -m "Core services"
jj bookmark create TASK-123/02-core-services

jj new TASK-123/02-core-services -m "API endpoints"
jj bookmark create TASK-123/03-api-endpoints

jj new TASK-123/03-api-endpoints -m "Tests"
jj bookmark create TASK-123/04-tests

# Step 2: Extract layers (bottom-up)

# Layer 01: Just abc123
jj new -B TASK-123/01-database-models --no-edit
jj restore --from abc123 models/
jj describe -m "Add User database model"
jj squash --into TASK-123/01-database-models

# Layer 02: def456 + mno345 (service + fix)
jj new -B TASK-123/02-core-services --no-edit
jj restore --from def456 services/
jj restore --from mno345 services/  # Overwrites with fixed version
jj describe -m "Implement authentication service

Includes typo fix from mno345"
jj squash --into TASK-123/02-core-services

# Layer 03: ghi789
jj new -B TASK-123/03-api-endpoints --no-edit
jj restore --from ghi789 api/
jj describe -m "Add authentication API routes"
jj squash --into TASK-123/03-api-endpoints

# Layer 04: jkl012
jj new -B TASK-123/04-tests --no-edit
jj restore --from jkl012 tests/
jj describe -m "Add test suite for authentication"
jj squash --into TASK-123/04-tests

# Step 3: Validate
jj log -r 'TASK-123/root::TASK-123/04-tests'
pytest  # All tests pass
```

### Example 2: Complex Reorganization

**Scenario:** 12 messy commits, mixed concerns, needs heavy reorganization.

```bash
# Original commits (abbreviated):
# c1: Add User model + start AuthService
# c2: WIP auth
# c3: Add API route for login
# c4: Add Session model
# c5: Update AuthService for sessions
# c6: Fix API route
# c7: Add logout route
# c8: Add tests
# c9: Fix session bug
# c10: Update requirements
# c11: Add registration endpoint
# c12: More tests

# Analysis: Complete mess, needs top-down reorganization

# Step 1: Bookmark original
jj bookmark create TASK-123/original

# Step 2: Build clean Layer 01 (Database)
jj new main -m "Database schema"
jj bookmark create TASK-123/01-database-schema

jj new -B TASK-123/01-database-schema --no-edit

# Extract all database stuff from various commits
jj restore --from c1 models/user.py
jj restore --from c4 models/session.py

# Manually add migrations (create clean versions)
cat > migrations/001_add_users.sql << 'EOF'
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT NOW()
);
EOF

cat > migrations/002_add_sessions.sql << 'EOF'
CREATE TABLE sessions (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id),
    token VARCHAR(255) UNIQUE NOT NULL,
    expires_at TIMESTAMP NOT NULL
);
EOF

# Update requirements
echo "sqlalchemy==2.0.0" > requirements.txt

jj describe -m "Add database schema for authentication

Tables:
- users: User accounts with email and password
- sessions: User sessions with tokens and expiry

Includes SQLAlchemy models for both tables."

jj squash --into TASK-123/01-database-schema

# Step 3: Build clean Layer 02 (Core Services)
jj new TASK-123/01-database-schema -m "Authentication services"
jj bookmark create TASK-123/02-auth-services

jj new -B TASK-123/02-auth-services --no-edit

# Extract and clean up service code from c1, c2, c5, c9
jj restore --from c1 services/auth_service.py
jj restore --from c5 services/auth_service.py
jj restore --from c9 services/auth_service.py  # Has bug fix

# Manually refactor to be clean
# Edit services/auth_service.py
# - Remove WIP code from c2
# - Integrate session support from c5
# - Include bug fix from c9
# - Organize into clear methods

# Add requirements
echo "bcrypt==4.0.0" >> requirements.txt

jj describe -m "Implement authentication and session services

AuthService:
- register_user(): Create new user with hashed password
- authenticate(): Verify credentials, create session
- validate_session(): Check session token validity
- logout(): Invalidate session

Includes fix for session expiration edge case."

jj squash --into TASK-123/02-auth-services

# Step 4: Build clean Layer 03 (API)
jj new TASK-123/02-auth-services -m "API endpoints"
jj bookmark create TASK-123/03-api-endpoints

jj new -B TASK-123/03-api-endpoints --no-edit

# Extract API code from c3, c6, c7, c11
jj restore --from c3 api/routes.py
jj restore --from c6 api/routes.py  # Has fix
jj restore --from c7 api/routes.py  # Has logout
jj restore --from c11 api/routes.py  # Has registration

# Refactor into organized structure
# Edit api/routes.py
# - Combine all endpoints cleanly
# - Fix applied from c6
# - Proper error handling

cat > api/schemas.py << 'EOF'
from pydantic import BaseModel, EmailStr

class UserRegister(BaseModel):
    email: EmailStr
    password: str

class UserLogin(BaseModel):
    email: EmailStr
    password: str

class SessionResponse(BaseModel):
    token: str
    expires_at: str
EOF

# Add requirements
echo "fastapi==0.100.0" >> requirements.txt
echo "pydantic[email]==2.0.0" >> requirements.txt

jj describe -m "Add authentication REST API

Endpoints:
- POST /auth/register: User registration
- POST /auth/login: User login (returns session token)
- POST /auth/logout: Session invalidation
- GET /auth/validate: Session validation

Includes Pydantic schemas for request/response validation."

jj squash --into TASK-123/03-api-endpoints

# Step 5: Build clean Layer 04 (Tests)
jj new TASK-123/03-api-endpoints -m "Test suite"
jj bookmark create TASK-123/04-tests

jj new -B TASK-123/04-tests --no-edit

# Extract tests from c8, c12
jj restore --from c8 tests/
jj restore --from c12 tests/

# Organize tests into proper structure
mkdir -p tests/unit tests/integration

# Edit and organize test files
# tests/unit/test_auth_service.py
# tests/integration/test_api.py

# Add requirements
echo "pytest==7.4.0" >> requirements.txt
echo "pytest-asyncio==0.21.0" >> requirements.txt

jj describe -m "Add comprehensive test suite

Unit Tests:
- test_auth_service.py: Core authentication logic
- test_session_manager.py: Session management

Integration Tests:
- test_api.py: End-to-end API testing

Coverage: 94%"

jj squash --into TASK-123/04-tests

# Step 6: Validate
jj log -r 'TASK-123/root::TASK-123/04-tests'

# Test each layer
jj new TASK-123/01-database-schema && alembic upgrade head
jj new TASK-123/02-auth-services && pytest tests/unit/
jj new TASK-123/04-tests && pytest

# Compare with original
jj diff -r TASK-123/original..TASK-123/04-tests
# Should show only improvements (cleaner code, better organization)

# Success! Clean stack from messy commits
```

---

## Troubleshooting

### Problem: Can't Figure Out Where Changes Came From

```bash
# Solution: Use jj log with file filtering

# Find all commits that touched specific file
jj log -r 'main..@original' --summary | grep -A 5 "auth_service.py"

# See full history of file
jj log -r 'file(services/auth_service.py) & main..@original'
```

### Problem: Moved Wrong Changes to Layer

```bash
# Solution: Use operation log to undo

jj op log
# Find operation before the wrong move

jj undo
# Or restore to specific operation
jj op restore <operation-id>

# Then redo correctly
```

### Problem: Lost Track of What's in Each Layer

```bash
# Solution: Review each layer systematically

# See what files are in each layer
for layer in 01-database-models 02-core-services 03-api-endpoints 04-tests; do
    echo "=== TASK-123/$layer ==="
    jj show TASK-123/$layer --summary
    echo
done

# See full diff of each layer
jj diff -r 'TASK-123/root..TASK-123/01-database-models'
jj diff -r 'TASK-123/01-database-models..TASK-123/02-core-services'
# etc.
```

### Problem: Tests Fail After Reorganization

```bash
# Solution: Identify what's different

# Run tests on original
jj new TASK-123/original
pytest -v > original-results.txt

# Run tests on converted
jj new TASK-123/04-tests
pytest -v > converted-results.txt

# Find what broke
diff original-results.txt converted-results.txt

# Common issues:
# 1. Import paths changed
# 2. Missing dependency in requirements.txt
# 3. File in wrong layer
# 4. Configuration files not copied

# Debug specific test
jj new TASK-123/04-tests
pytest tests/test_auth_service.py::test_login -vv
```

### Problem: Merge Conflicts During Reorganization

```bash
# Conflicts shouldn't happen during extraction (read-only)
# If you see conflicts, likely doing rebasing wrong

# Solution: Use restore/move instead of rebase

# BAD (can cause conflicts):
jj rebase -s old-commit -d TASK-123/01-database-models

# GOOD (no conflicts):
jj new -B TASK-123/01-database-models --no-edit
jj restore --from old-commit files/
jj squash --into TASK-123/01-database-models
```

### Problem: Circular Dependencies Between Layers

```bash
# Layer 02 needs something from Layer 03
# Layer 03 needs something from Layer 02

# Solution: Identify the real dependency order and refactor

# Example:
# api/routes.py imports services/auth_service.py (correct)
# services/auth_service.py imports api/schemas.py (wrong!)

# Fix: Move schemas to Layer 02 or extract shared schemas

jj new -B TASK-123/02-core-services --no-edit
# Move api/schemas.py to shared/schemas.py
jj squash --into TASK-123/02-core-services

# Update imports in Layer 03
jj new -B TASK-123/03-api-endpoints --no-edit
# Edit api/routes.py: import from shared.schemas instead
jj squash --into TASK-123/03-api-endpoints
```

---

## Best Practices

### 1. Always Backup Before Converting

```bash
# Before starting conversion
jj bookmark create TASK-123/backup
jj op log  # Note current operation ID

# If conversion goes wrong
jj bookmark set TASK-123/backup -r TASK-123/original
jj op restore <operation-id>
```

### 2. Convert in Small Batches

```bash
# Don't try to convert 50 commits at once

# Good approach:
# 1. Convert Layer 01 (database) - test it
# 2. Convert Layer 02 (services) - test it
# 3. Convert Layer 03 (API) - test it
# 4. Convert Layer 04 (tests) - test it

# After each layer, verify:
jj log -r 'TASK-123/root::TASK-123/XX-current-layer'
pytest
```

### 3. Test At Each Layer

```bash
# Don't wait until end to test

# After Layer 01:
jj new TASK-123/01-database-models
python -c "from models.user import User; print('OK')"

# After Layer 02:
jj new TASK-123/02-core-services
pytest tests/test_auth_service.py -v

# Catches problems early!
```

### 4. Write Clear Commit Messages for Layers

```bash
# Bad message:
jj describe -m "Layer 2"

# Good message:
jj describe -m "Implement authentication and session services

Core Components:
- AuthService: User registration and login with bcrypt
- SessionManager: Token-based session lifecycle
- PasswordUtils: Password hashing utilities

This layer provides the business logic for user authentication,
independent of any API or database implementation.

Note: Includes fix from original commit abc123 for empty password edge case."
```

### 5. Document Why Changes Were Reorganized

```bash
# Add notes about reorganization in descriptions

jj describe -m "Add authentication API endpoints

Reorganization notes:
- Combined original commits c3, c6, c7, c11
- Refactored to use consistent error handling
- Moved schema definitions to api/schemas.py
- Applied fix from c6 (session validation)

See TASK-123/original for original commit history."
```

### 6. Keep Original Branch Until Stack is Merged

```bash
# Don't delete original until all PRs merged

jj bookmark list | grep TASK-123
# TASK-123/original (keep this!)
# TASK-123/01-database-models
# TASK-123/02-core-services
# ...

# Only after all layers merged to main:
jj bookmark delete TASK-123/original
```

### 7. Use Descriptive Layer Names

```bash
# Bad:
TASK-123/01-part1
TASK-123/02-part2

# Good:
TASK-123/01-database-schema
TASK-123/02-auth-services
TASK-123/03-api-endpoints
TASK-123/04-test-coverage
```

### 8. Verify No Unintended Changes

```bash
# Before publishing stack, compare with original

jj diff -r TASK-123/original..TASK-123/04-tests > conversion-delta.patch

# Review the diff
cat conversion-delta.patch

# Should only show intentional improvements:
# - Better organization
# - Fixed bugs
# - Refactored code

# Should NOT show:
# - Lost functionality
# - Removed features
# - Missing files
```

### 9. Consider Incremental Conversion

```bash
# For very large branches, convert incrementally

# Week 1: Convert first 10 commits into Layer 01-02
# Publish and merge those PRs

# Week 2: Convert next 10 commits into Layer 03-04
# Build on already-merged layers

# Advantages:
# - Smaller chunks
# - Earlier feedback
# - Less risky
```

### 10. Use jj split for Mixed Commits

```bash
# When one commit has mixed concerns, split it

jj edit abc123  # Commit with database + API changes

jj split
# Interactive split:
# Select: migrations/, models/ → First commit
# Remaining: api/ → Second commit

# Now move each piece
jj squash --from <first-split> --into TASK-123/01-database-models migrations/ models/
jj squash --from <second-split> --into TASK-123/03-api-endpoints api/
```

---

## Summary

Converting an existing branch to stacked PRs requires:

**Analysis:**
- Review existing commits
- Identify file groupings
- Map commits to desired layers
- Identify dependencies

**Strategy Selection:**
- Bottom-Up: For mostly-clean commits
- Top-Down: For messy commits needing rewrite
- Surgical: For precise control

**Execution:**
- Create empty stack structure
- Extract/build each layer
- Handle special cases (splits, merges, refactors)
- Test each layer independently

**Validation:**
- Compare with original branch
- Run tests at each layer
- Verify no lost changes
- Check performance

**Key Commands:**
```bash
jj restore --from COMMIT files/     # Extract files from old commits
jj squash --from SRC --into DEST files/ # Move changes between commits
jj split                            # Split monolithic commits
jj squash --into BOOKMARK           # Build up layers
jj undo                          # Fix mistakes
```

**Remember:**
- Always backup original branch
- Test incrementally
- Write clear commit messages
- Document reorganization decisions
- Keep original until stack fully merged

Converting messy branches to clean stacked PRs is one of jj's superpowers! 🎯
