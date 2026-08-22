# 🔀 Git Cheatsheet

A practical, no-fluff Git cheatsheet for everyday development.

---

## 📦 Setup & Configuration

```bash
# Set your identity
git config --global user.name "Your Name"
git config --global user.email "you@example.com"

# Set default branch name
git config --global init.defaultBranch main

# Enable colored output
git config --global color.ui auto

# Set VS Code as default editor
git config --global core.editor "code --wait"

# View all config
git config --list
```

---

## 🚀 Creating Repositories

```bash
# Initialize a new repo
git init

# Clone an existing repo
git clone https://github.com/user/repo.git

# Clone into a specific folder
git clone https://github.com/user/repo.git my-folder

# Clone a specific branch
git clone -b branch-name https://github.com/user/repo.git
```

---

## 📝 Basic Workflow

```bash
# Check status
git status

# Stage specific files
git add file.py

# Stage all changes
git add .

# Stage parts of a file interactively
git add -p file.py

# Commit with message
git commit -m "feat: add login feature"

# Stage and commit in one step (tracked files only)
git commit -am "fix: resolve null pointer"

# Amend the last commit (change message or add files)
git commit --amend -m "updated message"
```

---

## 🌿 Branching

```bash
# List all branches
git branch          # local
git branch -r       # remote
git branch -a       # all

# Create a new branch
git branch feature-login

# Switch to a branch
git checkout feature-login
# or (modern way)
git switch feature-login

# Create and switch in one step
git checkout -b feature-login
# or
git switch -c feature-login

# Delete a branch (safe — only if merged)
git branch -d feature-login

# Force delete (even if not merged)
git branch -D feature-login

# Rename current branch
git branch -m new-name

# Delete a remote branch
git push origin --delete feature-login
```

---

## 🔄 Merging & Rebasing

```bash
# Merge a branch into current branch
git merge feature-login

# Merge with no fast-forward (preserves branch history)
git merge --no-ff feature-login

# Abort a merge in progress
git merge --abort

# Rebase current branch onto main
git rebase main

# Interactive rebase (squash, reorder, edit commits)
git rebase -i HEAD~3

# Abort a rebase
git rebase --abort

# Continue after resolving conflicts
git rebase --continue
```

---

## 🌐 Remote Repositories

```bash
# View remotes
git remote -v

# Add a remote
git remote add origin https://github.com/user/repo.git

# Change remote URL
git remote set-url origin https://github.com/user/new-repo.git

# Fetch changes (doesn't merge)
git fetch origin

# Pull changes (fetch + merge)
git pull origin main

# Pull with rebase (cleaner history)
git pull --rebase origin main

# Push changes
git push origin main

# Push and set upstream
git push -u origin feature-branch

# Force push (⚠️ careful!)
git push --force-with-lease origin branch
```

---

## 🔍 Inspecting & Comparing

```bash
# View commit history
git log
git log --oneline
git log --oneline --graph --all

# Show last N commits
git log -n 5

# View changes (unstaged)
git diff

# View staged changes
git diff --staged

# Compare branches
git diff main..feature-branch

# Show a specific commit
git show abc1234

# Search commit messages
git log --grep="login"

# Find who changed each line
git blame file.py

# Search for a string across all commits
git log -S "function_name" --all
```

---

## ⏪ Undoing Changes

```bash
# Discard changes in working directory
git checkout -- file.py
# or (modern)
git restore file.py

# Unstage a file (keep changes)
git reset HEAD file.py
# or (modern)
git restore --staged file.py

# Undo last commit (keep changes staged)
git reset --soft HEAD~1

# Undo last commit (keep changes unstaged)
git reset --mixed HEAD~1

# Undo last commit (⚠️ discard changes)
git reset --hard HEAD~1

# Create a new commit that undoes a specific commit
git revert abc1234

# Revert without auto-committing
git revert --no-commit abc1234
```

---

## 📦 Stashing

```bash
# Stash current changes
git stash

# Stash with a message
git stash save "work in progress on login"

# List stashes
git stash list

# Apply most recent stash (keep in stash list)
git stash apply

# Apply and remove from stash list
git stash pop

# Apply a specific stash
git stash apply stash@{2}

# Drop a specific stash
git stash drop stash@{0}

# Clear all stashes
git stash clear
```

---

## 🏷️ Tags

```bash
# List tags
git tag

# Create a lightweight tag
git tag v1.0.0

# Create an annotated tag (recommended)
git tag -a v1.0.0 -m "Release version 1.0.0"

# Push a specific tag
git push origin v1.0.0

# Push all tags
git push origin --tags

# Delete a local tag
git tag -d v1.0.0

# Delete a remote tag
git push origin --delete v1.0.0
```

---

## 🧹 Cleanup

```bash
# Remove untracked files (dry run)
git clean -n

# Remove untracked files (for real)
git clean -f

# Remove untracked files and directories
git clean -fd

# Garbage collect (optimize repo)
git gc

# Remove branches that no longer exist on remote
git fetch --prune
```

---

## 💡 Useful Aliases

Add these to your `~/.gitconfig`:

```ini
[alias]
    s = status
    co = checkout
    br = branch
    ci = commit
    lg = log --oneline --graph --all --decorate
    last = log -1 HEAD
    unstage = reset HEAD --
    undo = reset --soft HEAD~1
    amend = commit --amend --no-edit
    wip = !git add -A && git commit -m "WIP"
```

---

## 📋 Commit Message Convention

Follow the [Conventional Commits](https://www.conventionalcommits.org/) format:

```
<type>(<scope>): <description>

Types:
  feat:     New feature
  fix:      Bug fix
  docs:     Documentation changes
  style:    Code style (formatting, semicolons)
  refactor: Code refactoring
  test:     Adding or updating tests
  chore:    Maintenance tasks
  perf:     Performance improvements
  ci:       CI/CD changes
```

**Examples:**
```
feat(auth): add Google OAuth login
fix(api): handle null response from /users endpoint
docs(readme): update installation instructions
```

---

## 🔑 SSH Key Setup

```bash
# Generate SSH key
ssh-keygen -t ed25519 -C "you@example.com"

# Start SSH agent
eval "$(ssh-agent -s)"

# Add key to agent
ssh-add ~/.ssh/id_ed25519

# Copy public key (macOS)
pbcopy < ~/.ssh/id_ed25519.pub

# Test connection
ssh -T git@github.com
```

---

*Made with ❤️ for developers who forget Git commands (all of us).*
