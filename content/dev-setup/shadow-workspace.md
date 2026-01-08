---
id: shadow-workspace
type: dev-setup
title: "Shadow Workspace"
---

# Shadow Workspace

Work safely with AI-generated changes using a shadow copy of your project.

## Overview

A shadow workspace lets you:

- **Review AI changes** before they touch your real code
- **Test in isolation** without risk to your project
- **Compare diffs** between original and modified
- **Rollback easily** if something goes wrong

## The Problem

When Amplifier writes code, it modifies files directly. This can be risky:

- Changes might break things
- Hard to see what changed
- Difficult to undo

## The Solution

Work in a shadow copy:

```
my-project/           # Your real project (read-only to AI)
my-project-shadow/    # Shadow copy (AI writes here)
```

## Setup Methods

### Method 1: Manual Copy

```bash
# Create shadow workspace
cp -r my-project my-project-shadow
cd my-project-shadow

# Start Amplifier
amp
```

### Method 2: Git Worktree

If your project uses git:

```bash
cd my-project

# Create worktree
git worktree add ../my-project-shadow -b ai-changes

# Work in shadow
cd ../my-project-shadow
amp
```

Benefits:
- Shares git history
- Easy to create PR
- Clean branch management

### Method 3: bkrabach's Shadow Script

From bkrabach's setup-tools:

```bash
# Install
curl -o ~/bin/shadow https://raw.githubusercontent.com/bkrabach/setup-tools/main/shadow.sh
chmod +x ~/bin/shadow

# Use
shadow my-project
# Creates my-project-shadow and cd's into it
```

## Workflow

### Step 1: Create Shadow

```bash
shadow my-project
# or
cp -r my-project my-project-shadow
cd my-project-shadow
```

### Step 2: Work with Amplifier

```bash
amp

> Refactor the authentication module
> Add comprehensive tests
> Update the documentation
```

### Step 3: Review Changes

```bash
# See what changed
diff -r ../my-project . --exclude=.git --exclude=node_modules

# Or with git
git diff
git status
```

### Step 4: Accept or Reject

**Accept changes:**
```bash
# If using git worktree
git add .
git commit -m "AI-assisted refactoring"
git checkout main
git merge ai-changes

# If manual copy
cp -r . ../my-project
```

**Reject changes:**
```bash
# Just delete the shadow
cd ..
rm -rf my-project-shadow
```

## Best Practices

### Keep Shadow Fresh

Sync periodically:
```bash
cd my-project-shadow
rsync -av --exclude='.git' ../my-project/ .
```

### Use Git Worktrees for Teams

```bash
# Create feature branch worktree
git worktree add ../feature-auth -b feature/auth-refactor

# Work with AI
cd ../feature-auth
amp

# Create PR when done
gh pr create
```

### Exclude Large Directories

When copying:
```bash
rsync -av --exclude='node_modules' --exclude='.venv' \
  --exclude='dist' --exclude='build' \
  my-project/ my-project-shadow/
```

### Multiple Shadows

For comparing approaches:
```bash
shadow my-project approach-a
shadow my-project approach-b

# Try different prompts in each
cd ../approach-a && amp
cd ../approach-b && amp

# Compare results
diff -r approach-a approach-b
```

## Shadow Script

Create `~/bin/shadow`:

```bash
#!/bin/bash
# Shadow workspace creator

if [ -z "$1" ]; then
  echo "Usage: shadow <project-dir> [shadow-name]"
  exit 1
fi

PROJECT="$1"
SHADOW="${2:-$1-shadow}"

if [ -d "$SHADOW" ]; then
  echo "Shadow already exists: $SHADOW"
  cd "$SHADOW"
else
  echo "Creating shadow workspace: $SHADOW"
  
  if [ -d "$PROJECT/.git" ]; then
    # Use git worktree if possible
    cd "$PROJECT"
    git worktree add "../$SHADOW" -b "shadow-$(date +%Y%m%d-%H%M%S)"
    cd "../$SHADOW"
  else
    # Fall back to copy
    cp -r "$PROJECT" "$SHADOW"
    cd "$SHADOW"
  fi
  
  echo "Shadow workspace ready: $SHADOW"
fi

# Start shell in shadow
exec $SHELL
```

## Try It Yourself

### Exercise 1: Create Shadow

```bash
# Pick any project
shadow my-project

# Verify you're in shadow
pwd
```

### Exercise 2: Make AI Changes

```bash
amp

> Add input validation to all API endpoints
```

### Exercise 3: Review and Decide

```bash
# See changes
git diff

# Accept or reject
```

## Troubleshooting

### "Directory already exists"

```bash
rm -rf my-project-shadow
shadow my-project
```

### Large Projects Take Too Long

Use rsync with exclusions:
```bash
rsync -av --exclude='node_modules' --exclude='.venv' \
  --progress my-project/ my-project-shadow/
```

### Git Worktree Conflicts

```bash
# Clean up orphaned worktree
git worktree prune
git worktree list
```

## Next

Learn about remote development options:

→ [Remote Development](remote-dev.md)
