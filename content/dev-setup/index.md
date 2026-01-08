---
id: dev-setup-overview
type: dev-setup
title: "Developer Setup"
---

# Developer Setup

Tools and techniques for productive Amplifier development.

## Overview

This section covers:

| Topic | What You'll Learn |
|-------|-------------------|
| [CLI Tools](cli-tools.md) | Essential command-line utilities |
| [Shadow Workspace](shadow-workspace.md) | Safe AI-assisted editing |
| [Remote Development](remote-dev.md) | Docker, SSH, Codespaces |
| [Debugging](debugging.md) | Troubleshooting techniques |

## Quick Setup Checklist

### 1. Essential Tools

```bash
# Python package manager
curl -LsSf https://astral.sh/uv/install.sh | sh

# Amplifier
uv tool install amplifier

# Helpful utilities
brew install jq fzf bat ripgrep  # macOS
# or apt install equivalent
```

### 2. Provider Configuration

```bash
# Pick your provider
export ANTHROPIC_API_KEY="sk-ant-..."
# or
export OPENAI_API_KEY="sk-..."

# Make permanent
echo 'export ANTHROPIC_API_KEY="your-key"' >> ~/.bashrc
```

### 3. Shell Aliases

Add to `~/.bashrc` or `~/.zshrc`:

```bash
alias a='amp'
alias ar='amp run'
alias as='amp session'
```

### 4. Verify Setup

```bash
amp run "Hello! What model are you?"
```

## Development Patterns

### Pattern 1: Shadow Workspace

For safe AI-assisted editing:

```bash
# Create shadow copy
cp -r my-project my-project-shadow
cd my-project-shadow

# Work with AI
amp

# Review changes
diff -r ../my-project .

# Accept or discard
```

### Pattern 2: Git Worktree

For version-controlled changes:

```bash
cd my-project
git worktree add ../ai-changes -b feature/ai-assist

cd ../ai-changes
amp

# Create PR when done
gh pr create
```

### Pattern 3: Container Isolation

For complete isolation:

```bash
docker run -it --rm \
  -v $(pwd):/workspace \
  -e ANTHROPIC_API_KEY \
  amplifier-dev
```

## Productivity Tips

### 1. Use Session History

```bash
# Resume last session
amp session list
amp session resume [id]
```

### 2. Leverage Specialists

```
> Use bug-hunter to debug this
> Use zen-architect to design this
> Use explorer to understand this codebase
```

### 3. Save Common Prompts

Create a `prompts/` directory:

```
prompts/
├── code-review.md
├── refactor.md
└── test-coverage.md
```

```bash
amp run "$(cat prompts/code-review.md) - review src/auth.py"
```

### 4. Use Recipes for Repeatable Tasks

```yaml
# .amplifier/recipes/review.yaml
name: code-review
steps:
  - id: review
    instruction: "Review {{file}} for issues"
```

```bash
amp recipes execute .amplifier/recipes/review.yaml \
  --context '{"file": "src/main.py"}'
```

## Troubleshooting Quick Reference

| Problem | Solution |
|---------|----------|
| "Command not found" | Check PATH, reinstall |
| "API key invalid" | Verify environment variable |
| "Context too long" | Start new session |
| "Tool failed" | Check permissions, paths |
| "Session won't resume" | Use session-analyst |

## Recommended Reading Order

1. **[CLI Tools](cli-tools.md)** - Set up your environment
2. **[Shadow Workspace](shadow-workspace.md)** - Work safely
3. **[Remote Development](remote-dev.md)** - Scale up
4. **[Debugging](debugging.md)** - Fix issues

Each guide includes exercises to practice the concepts.
