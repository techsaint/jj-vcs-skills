# Jujutsu (jj) Setup Guide

A comprehensive guide to installing, configuring, and setting up Jujutsu (jj) for daily use.

## Table of Contents

1. [Installation](#installation)
2. [Initial Configuration](#initial-configuration)
3. [Git Integration](#git-integration)
4. [Aliases and Shortcuts](#aliases-and-shortcuts)
5. [Editor Integration](#editor-integration)
6. [Shell Integration](#shell-integration)
7. [Diff and Merge Tools](#diff-and-merge-tools)
8. [Templates](#templates)
9. [Authentication](#authentication)
10. [Migration Strategies](#migration-strategies)

---

## Installation

### Linux

#### From Source (Rust/Cargo)
Requires Rust 1.88+ and build-essential:

```bash
# Install dependencies
sudo apt-get install build-essential

# Install latest release
cargo install --locked --bin jj jj-cli

# OR install prerelease (main branch)
cargo install --git https://github.com/jj-vcs/jj.git --locked --bin jj jj-cli
```

#### Arch Linux
```bash
# Official repository
pacman -S jujutsu

# AUR (development version)
yay -S jujutsu-git
```

#### NixOS
```bash
# Released version (add to configuration.nix or install to profile)
nix profile install nixpkgs#jujutsu

# Prerelease version (from flake)
nix profile install 'github:jj-vcs/jj'

# Run without installing
nix run 'github:jj-vcs/jj'
```

#### Homebrew (Linux)
```bash
brew install jj
```

#### Gentoo Linux
```bash
# Enable GURU repository first
emerge -av dev-vcs/jj
```

#### openSUSE Tumbleweed
```bash
zypper install jujutsu
```

### macOS

#### Homebrew
```bash
brew install jj
```

#### MacPorts
```bash
sudo port install jujutsu
```

#### From Source
```bash
# Install Xcode command line tools
xcode-select --install

# Install via cargo
cargo install --locked --bin jj jj-cli
```

### Windows

#### Via Cargo
```bash
# Requires Rust 1.88+
cargo install --locked --bin jj jj-cli
```

#### Via winget
```bash
winget install jj-vcs.jj
```

#### Via scoop
```bash
scoop install main/jj
```

#### Pre-built Binaries
Download from [GitHub Releases](https://github.com/jj-vcs/jj/releases/latest)

### Cargo Binstall
If you use `cargo-binstall`:

```bash
cargo binstall --strategies crate-meta-data jj-cli
```

---

## Initial Configuration

### Required: User Identity

Set your name and email (required for commits):

```bash
jj config set --user user.name "Your Name"
jj config set --user user.email "your.email@example.com"
```

### Configuration File Location

Find your config file:

```bash
jj config path --user
```

Default locations:
- **Linux/macOS**: `~/.config/jj/config.toml` or `~/.jjconfig.toml`
- **Windows**: `%APPDATA%\jj\config.toml`

### Configuration File Structure

jj uses TOML format with optional JSON Schema validation. Add this at the top of your config files for editor validation:

```toml
#:schema https://docs.jj-vcs.dev/latest/config-schema.json

[user]
name = "Your Name"
email = "your.email@example.com"
```

### Configuration Hierarchy

Configuration is loaded in this order (later overrides earlier):
1. Built-in defaults
2. User config (`~/.config/jj/config.toml`)
3. User config directory (`~/.config/jj/conf.d/*.toml` - loaded alphabetically)
4. Repo config (`.jj/repo/config.toml`)
5. Workspace config (`.jj/workspace-config.toml`)
6. Command-line (`--config` flags)

---

## Git Integration

### Creating a New Repository

```bash
# Create a new Git-backed jj repository (colocated by default)
jj git init my-project
cd my-project

# Non-colocated repository (Git repo hidden in .jj/)
jj git init --no-colocate my-project
```

### Cloning a Git Repository

```bash
# Clone from remote (creates colocated workspace by default)
jj git clone https://github.com/user/repo.git

# With custom remote name
jj git clone --remote upstream https://github.com/user/repo.git

# Non-colocated clone
jj git clone --no-colocate https://github.com/user/repo.git
```

### Working with Existing Git Repos

```bash
# Create jj workspace from existing Git repo
jj git init --git-repo=/path/to/existing/repo my-workspace
```

### Colocated vs Non-Colocated Workspaces

**Colocated** (default): `.git` and `.jj` directories side-by-side
- Pros: Can use git and jj commands interchangeably
- Cons: Slower in large repos, potential conflicts if mixing tools

**Non-colocated**: Git repo hidden inside `.jj/repo/store/`
- Pros: Faster, cleaner separation
- Cons: Explicit import/export needed with `jj git import/export`

```toml
# Disable colocation by default
[git]
colocate = false
```

### Converting Between Modes

```bash
# Check colocation status
jj git colocation status

# Enable colocation
jj git colocation enable

# Disable colocation
jj git colocation disable
```

### Git Remotes Configuration

```bash
# Set default fetch remote(s)
jj config set --repo git.fetch "upstream"
jj config set --repo git.fetch '["origin", "upstream"]'

# Set default push remote
jj config set --repo git.push "origin"

# Auto-track bookmarks (branches) with pattern
jj config set --repo remotes.origin.auto-track-bookmarks "glob:*"
```

Example configuration for a GitHub fork workflow:

```toml
[remotes.origin]
auto-track-bookmarks = "glob:*"

[remotes.upstream]
auto-track-bookmarks = "main"
```

---

## Aliases and Shortcuts

### Defining Command Aliases

```toml
[aliases]
# Simple command aliases
st = ["status"]
l = ["log", "-r", "(main..@):: | (main..@)-"]

# Show my recent work
recent = ["log", "-r", "mine() & ::@", "-n", "20"]

# Commit and push
cp = ["commit", "--push"]

# Interactive split with meld
spliti = ["split", "--tool", "meld"]
```

### Running External Commands

Use `jj util exec` to run scripts (be cautious!):

```toml
[aliases]
# Run custom script
my-script = ["util", "exec", "--", "my-jj-script"]

# Inline bash script
cleanup = ["util", "exec", "--", "bash", "-c", """
  set -euo pipefail
  echo "Cleaning up..."
  # Your script here
""", ""]
```

### Setting Default Command

```toml
[ui]
# Command to run when no subcommand specified
default-command = ["log", "--reversed"]
```

---

## Editor Integration

### Setting Your Editor

Priority order: `$JJ_EDITOR` > `ui.editor` > `$VISUAL` > `$EDITOR`

```toml
[ui]
# Terminal editors
editor = "nvim"
editor = "vim"
editor = "nano"
editor = "emacs"

# GUI editors (usually need --wait flag)
editor = "code -w"                    # VS Code
editor = "code.cmd -w"                # VS Code (Windows)
editor = "subl -n -w"                 # Sublime Text
editor = "mate -w"                    # TextMate
editor = "idea --temp-project --wait" # IntelliJ

# Notepad++ (Windows)
editor = ["C:/Program Files/Notepad++/notepad++.exe",
          "-multiInst", "-notabbar", "-nosession", "-noPlugin"]
```

### Environment Variables

```bash
# Set editor via environment variable
export JJ_EDITOR="nvim"
export VISUAL="code -w"
export EDITOR="nano"
```

---

## Shell Integration

### Command-Line Completion

jj provides two types of completions:
- **Standard**: Basic completions for subcommands and options
- **Dynamic**: Context-aware completions (recommended)

#### Bash

```bash
# Standard completions
echo 'source <(jj util completion bash)' >> ~/.bashrc

# Dynamic completions (recommended)
echo 'source <(COMPLETE=bash jj)' >> ~/.bashrc
```

#### Zsh

```bash
# Standard completions
cat >> ~/.zshrc << 'EOF'
autoload -U compinit
compinit
source <(jj util completion zsh)
EOF

# Dynamic completions (recommended)
echo 'source <(COMPLETE=zsh jj)' >> ~/.zshrc
```

#### Fish

Fish 4.0.2+ loads dynamic completions automatically. For older versions:

```fish
# Standard
jj util completion fish | source

# Dynamic (recommended)
COMPLETE=fish jj | source

# Add to config.fish for persistence
```

#### Nushell

```nushell
jj util completion nushell | save -f completions-jj.nu
use completions-jj.nu *
```

#### PowerShell

```powershell
# Standard
jj util completion power-shell | Out-String | Invoke-Expression

# Dynamic (recommended)
$env:COMPLETE = "powershell"
jj | Out-String | Invoke-Expression
Remove-Item Env:\COMPLETE

# Add to $PROFILE for persistence
```

Note: Set execution policy on Windows: `Set-ExecutionPolicy RemoteSigned -Scope CurrentUser`

### Shell Prompt Integration

Example for showing current change ID in prompt:

```bash
# Bash/Zsh prompt function
jj_prompt() {
  local change_id=$(jj log -r @ --no-graph --color=never -T 'change_id.short()' 2>/dev/null)
  if [ -n "$change_id" ]; then
    echo " (jj:$change_id)"
  fi
}

# Add to PS1/PROMPT
PS1='[\u@\h \W$(jj_prompt)]\$ '
```

---

## Diff and Merge Tools

### Diff Editors

#### Built-in Diff Editor (Default)

```toml
[ui]
diff-editor = ":builtin"  # Default, no external tool needed
```

#### Meld (Recommended External Tool)

```bash
# Install Meld
# Linux: sudo apt install meld
# macOS: brew install --cask meld
# Windows: Download from meldmerge.org
```

```toml
[ui]
# 2-pane view
diff-editor = "meld"

# 3-pane view (experimental, shows left/right/editing pane)
diff-editor = "meld-3"

# Custom configuration
[merge-tools.meld]
program = "/path/to/meld"  # If not in PATH
edit-args = ["--newtab", "$left", "$right"]
```

#### diffedit3

Easy to install, works over SSH:

```toml
[ui]
# 3-pane web-based diff editor
diff-editor = "diffedit3"

# For SSH use (doesn't auto-open browser)
diff-editor = "diffedit3-ssh"
```

For SSH port forwarding:
```bash
ssh -L 17376:localhost:17376 \
    -L 17377:localhost:17377 \
    -L 17378:localhost:17378 \
    -L 17379:localhost:17379 \
    -L 17380:localhost:17380 \
    myhost
```

#### Vim/Neovim

```toml
[ui]
diff-editor = "vimdiff"

[merge-tools.vimdiff]
diff-invocation-mode = "file-by-file"
```

Better: Use [DirDiff plugin](https://github.com/will133/vim-dirdiff) or [vimtabdiff](https://github.com/balki/vimtabdiff)

#### External Diff Tools

```toml
[ui]
# Difftastic
diff-editor = ["difft", "--color=always", "$left", "$right"]

# Delta (requires git-style diffs)
diff-editor = "delta"
diff-formatter = ":git"  # Use git-style diffs

[merge-tools.delta]
diff-args = ["--color=always", "$left", "$right"]
diff-expected-exit-codes = [0, 1]
```

### Merge Tools (Conflict Resolution)

#### Out-of-the-box Tools

These work automatically if installed:
- `meld`
- `kdiff3`
- `vimdiff`
- `vscode` / `vscodium`
- `smerge` (Sublime Merge)
- `mergiraf`

```toml
[ui]
merge-editor = "meld"      # Or kdiff3, vscode, etc.
```

#### VS Code

```toml
[ui]
merge-editor = "vscode"

# Works with VS Code Remote Development
```

#### Custom Merge Tool

```toml
[merge-tools.my-tool]
program = "/path/to/tool"
merge-args = ["$base", "$left", "$right", "-o", "$output"]

# If tool edits conflict markers instead of resolving
merge-tool-edits-conflict-markers = true

# Expected exit codes (besides 0) that indicate partial resolution
merge-conflict-exit-codes = [1]
```

#### Conflict Marker Style

```toml
[ui]
# "diff" (default): Shows diffs to apply
# "snapshot": Shows snapshot for each side
# "git": Git's diff3 format
conflict-marker-style = "diff"
```

### Diff/Merge Tool Variables

Tools can use these variables in their arguments:
- `$left`, `$right`: Paths to left/right sides
- `$base`: Path to common ancestor
- `$output`: Where to write result (merge only)
- `$path`: Repository-relative path of file
- `$marker_length`: Conflict marker length
- `$width`: Terminal width

---

## Templates

### Customizing Log Output

```toml
[templates]
# Full commit description in log
log = "builtin_log_compact_full_description"

# Custom log template
log = '''
separate(" ",
  change_id.short(),
  if(current_working_copy, "@"),
  bookmarks,
  tags,
  commit_id.short(),
  description.first_line(),
)
'''
```

### Commit ID Display

```toml
[template-aliases]
# Show at least 12 characters with unique prefix highlighted
'format_short_id(id)' = 'id.shortest(12)'

# Shortest unique prefix only
'format_short_id(id)' = 'id.shortest()'

# Always 12 characters
'format_short_id(id)' = 'id.short(12)'

# Uppercase change IDs
'format_short_change_id(id)' = 'format_short_id(id).upper()'
```

### Timestamp Formatting

```toml
[template-aliases]
# Relative timestamps ("2 hours ago")
'format_timestamp(timestamp)' = 'timestamp.ago()'

# ISO 8601 format
'format_timestamp(timestamp)' = 'timestamp'

# Custom format
'format_timestamp(timestamp)' = 'timestamp.format("%Y-%m-%d %H:%M")'
```

### Author Display

```toml
[template-aliases]
# Email only (default)
'format_short_signature(signature)' = 'signature.email()'

# Name and email
'format_short_signature(signature)' = 'signature'

# Username part only
'format_short_signature(signature)' = 'signature.email().local()'
```

### Commit Trailers

```toml
[templates]
commit_trailers = '''
format_signed_off_by_trailer(self)
++ if(!trailers.contains_key("Change-Id"),
       format_gerrit_change_id_trailer(self))
'''
```

### Default Commit Description

```toml
[templates]
draft_commit_description = '''
concat(
  builtin_draft_commit_description,
  "\nJJ: ignore-rest\n",
  diff.git(),
)
'''
```

---

## Authentication

jj uses git for remote operations, so authentication works the same way.

### SSH Keys

```bash
# Generate SSH key if needed
ssh-keygen -t ed25519 -C "your.email@example.com"

# Start SSH agent
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519

# Add public key to GitHub/GitLab/etc.
cat ~/.ssh/id_ed25519.pub
```

### HTTPS with Credential Helper

```bash
# Configure Git credential helper
git config --global credential.helper store
# Or on macOS: git config --global credential.helper osxkeychain
# Or on Windows: git config --global credential.helper manager-core

# First push/pull will prompt for credentials
jj git fetch
```

### Commit Signing

#### GPG Signing

```toml
[signing]
behavior = "own"  # Sign your own commits
backend = "gpg"
key = "4ED556E9729E000F"  # Or email, or omit to use user.email

[signing.backends.gpg]
program = "gpg2"  # Optional
allow-expired-keys = false
```

#### SSH Signing

```toml
[signing]
behavior = "own"
backend = "ssh"
key = "ssh-ed25519 AAAAC3Nza... your-key-here"
# Or: key = "~/.ssh/id_for_signing.pub"

[signing.backends.ssh]
program = "/path/to/ssh-keygen"  # Optional
allowed-signers = "/path/to/allowed-signers"
revocation-list = "/path/to/revocation-list"  # Optional
```

Allowed signers file format:
```
your.email@example.com ssh-ed25519 AAAAC3Nza...
```

#### Sign on Push Only

```toml
[signing]
behavior = "drop"  # Don't auto-sign during regular operations
backend = "ssh"
key = "~/.ssh/id_signing.pub"

[git]
sign-on-push = true  # Sign all commits when pushing
```

---

## Migration Strategies

### From Git to jj

#### Trial Run (Keep Using Git)

```bash
# Clone with jj, but continue using git
cd ~/my-git-repo
jj git init --git-repo=. jj-workspace
cd jj-workspace

# Use jj for some operations, git for others
# Both repos stay in sync
```

#### Full Migration (Colocated)

```bash
# Inside your existing Git repo
jj git init --colocate

# Now .jj/ and .git/ coexist
# Can use both jj and git commands
```

#### Full Migration (Separate)

```bash
# Create jj workspace pointing to existing Git repo
cd ~/projects
jj git init --git-repo=~/my-git-repo my-jj-workspace
cd my-jj-workspace

# Import changes from Git
jj git import

# Make changes with jj
jj new -m "My changes"

# Export to Git when needed
jj git export
```

### Key Workflow Differences

#### No Staging Area
```bash
# Git workflow
git add file.txt
git commit -m "Update file"

# jj workflow (automatic)
# Edit file.txt
jj commit -m "Update file"  # Or just let auto-amend happen
```

#### Working Copy is a Commit
```bash
# Git: working directory is separate from commits
# jj: working copy IS a commit that auto-amends

# See current changes
jj diff

# Start new change
jj new

# Go back to old change
jj edit <change-id>
```

#### Branches are Anonymous
```bash
# Git: must create branch
git checkout -b my-feature

# jj: changes exist without branches (bookmarks)
jj new main -m "My feature"
# Create bookmark only when pushing
jj bookmark create my-feature
```

### Import History from Git

```bash
# All Git history comes automatically
jj git clone https://github.com/user/repo.git

# See imported commits
jj log

# Git tags are imported as tags
# Git branches are imported as bookmarks
```

### Gradual Adoption Strategy

1. **Week 1-2: Read-only operations**
   ```bash
   jj log
   jj diff
   jj show
   ```

2. **Week 3-4: Basic modifications**
   ```bash
   jj new
   jj describe
   jj commit
   ```

3. **Week 5-6: Advanced operations**
   ```bash
   jj squash
   jj split
   jj rebase
   jj resolve
   ```

4. **Week 7+: Team collaboration**
   ```bash
   jj bookmark create
   jj git push
   jj git fetch
   ```

---

## Additional Configuration Tips

### Performance Optimization

```toml
# Use filesystem monitor for large repos
[fsmonitor]
backend = "watchman"

[fsmonitor.watchman]
register-snapshot-trigger = true

# Limit auto-tracked files
[snapshot]
auto-track = "glob:**/*.{rs,py,js,ts}"  # Example pattern
max-new-file-size = "10MiB"
```

### UI Customization

```toml
[ui]
# Color output
color = "auto"  # "always", "never", "auto"

# Pager
pager = ":builtin"  # Or "less -FRX", "delta", etc.
paginate = "auto"   # "always", "never", "auto"

# Graph style
graph.style = "curved"  # "square", "ascii", "ascii-large"

# Word wrapping in log
log-word-wrap = true
```

### Git Behavior

```toml
[git]
# Abandon unreachable commits when importing
abandon-unreachable-commits = true

# Private commits (won't push)
private-commits = "description(glob:'wip:*') | description(glob:'private:*')"

# Colocate by default
colocate = true

# Auto-track default bookmark on clone
track-default-bookmark-on-clone = true
```

### Code Formatting

```toml
# Auto-format on snapshot
[fix.tools.rustfmt]
command = ["rustfmt", "--emit", "stdout"]
patterns = ["glob:'**/*.rs'"]

[fix.tools.prettier]
command = ["prettier", "--stdin-filepath=$path"]
patterns = ["glob:'**/*.{js,ts,json,md}'"]
```

---

## Quick Reference Card

### Essential Commands

```bash
# Setup
jj git clone <url>              # Clone repository
jj git init                     # Initialize new repo
jj config set --user user.name  # Set identity

# Daily workflow
jj status                       # Show status (or: jj st)
jj log                          # Show log
jj diff                         # Show changes
jj new                          # Start new change
jj commit -m "message"          # Commit current change
jj describe -m "message"        # Set commit message

# Moving around
jj new <change>                 # Create change on top of
jj edit <change>                # Resume editing change

# Modifying history
jj squash                       # Move changes to parent
jj split                        # Split change in two
jj rebase -s <src> -d <dest>    # Rebase changes

# Branches (bookmarks)
jj bookmark create <name>       # Create bookmark
jj bookmark list                # List bookmarks
jj bookmark set <name> -r <rev> # Move bookmark

# Syncing
jj git fetch                    # Fetch from remote
jj git push                     # Push to remote
jj git push --change <change>   # Push specific change

# Undo
jj undo                         # Undo last operation
jj op log                       # View operation log
jj op restore <operation>       # Restore to operation

# Conflicts
jj resolve                      # Resolve conflicts
jj diff -r <change>             # View conflict
```

### Configuration Files

- User: `~/.config/jj/config.toml`
- Repo: `.jj/repo/config.toml`
- Workspace: `.jj/workspace-config.toml`

### Helpful Resources

- Official docs: https://docs.jj-vcs.dev/
- GitHub: https://github.com/jj-vcs/jj
- Discord: https://discord.gg/dkmfj3aGQN
- Tutorial: https://steveklabnik.github.io/jujutsu-tutorial/

---

## Troubleshooting

### Common Issues

**"No repo found"**
```bash
# Make sure you're in a jj repository
jj git init  # Or jj git clone
```

**Slow performance**
```bash
# Try garbage collection
jj util gc

# Enable filesystem monitor (see Performance section)
```

**Conflicts after operations**
```bash
# This is normal! jj preserves conflicts
jj resolve
# Or edit conflict markers manually
```

**Git and jj out of sync (colocated)**
```bash
# Import Git changes
jj git import

# Export jj changes
jj git export
```

**Can't push to remote**
```bash
# Make sure bookmark exists
jj bookmark create main

# Check remote configuration
jj bookmark list --all-remotes
```

### Getting Help

```bash
jj help                    # General help
jj help <command>          # Command help
jj <command> --help        # Brief help
jj help -k <keyword>       # Search help
```

---

This guide covers the essentials of setting up jj for daily use. For more detailed information, consult the [official documentation](https://docs.jj-vcs.dev/).
