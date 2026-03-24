---
id: cli-tools
type: dev-setup
title: "CLI Tools"
---

# CLI Tools

Useful command-line tools for Amplifier development.

## Overview

These tools enhance your Amplifier development workflow:

| Tool | Purpose |
|------|---------|
| `amp` | Main Amplifier CLI |
| `uv` | Fast Python package manager |
| `jq` | JSON processor |
| `fzf` | Fuzzy finder |
| `bat` | Better `cat` |
| `ripgrep` | Fast search |

## amp (Amplifier CLI)

### Installation

```bash
# Via uv (recommended)
uv tool install amplifier

# Verify
amp --version
```

### Key Commands

```bash
# Start interactive session
amp

# Run single prompt
amp run "explain this code"

# Use specific bundle
amp --bundle recipes

# List sessions
amp session list

# Resume session
amp session resume [id]
```

### Bundle Management

```bash
# Add a bundle
amp bundle add git+https://github.com/microsoft/amplifier-bundle-recipes@main

# List bundles
amp bundle list

# Use a bundle
amp bundle use recipes

# Remove a bundle
amp bundle remove recipes
```

### Recipe Commands

```bash
# Run recipe
amp recipes execute recipe.yaml --context '{}'

# List sessions
amp recipes list

# Resume recipe
amp recipes resume [session-id]

# Validate recipe
amp recipes validate recipe.yaml
```

## uv (Python Package Manager)

Fast, modern Python package management.

### Installation

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

### Key Commands

```bash
# Create virtual environment
uv venv

# Install packages
uv pip install requests

# Install from requirements
uv pip install -r requirements.txt

# Install tool globally
uv tool install ruff
```

### Why uv?

- **10-100x faster** than pip
- **Automatic venv** detection
- **Lockfile support**
- **Tool management**

## jq (JSON Processor)

Query and manipulate JSON from the command line.

### Installation

```bash
# macOS
brew install jq

# Ubuntu/Debian
sudo apt install jq

# Windows
winget install jqlang.jq
```

### Common Uses

```bash
# Pretty print JSON
curl https://api.example.com/users | jq .

# Extract field
cat data.json | jq '.name'

# Filter array
cat users.json | jq '.[] | select(.active == true)'

# Count items
cat items.json | jq 'length'

# Transform data
cat users.json | jq '{count: length, names: [.[].name]}'
```

### With Amplifier Logs

```bash
# Parse session events
cat events.jsonl | jq 'select(.event_type == "tool:call")'

# Get tool names used
cat events.jsonl | jq -r 'select(.event_type == "tool:call") | .tool_name'
```

## fzf (Fuzzy Finder)

Interactive fuzzy search for anything.

### Installation

```bash
# macOS
brew install fzf

# Ubuntu/Debian
sudo apt install fzf

# Setup shell integration
$(brew --prefix)/opt/fzf/install
```

### Common Uses

```bash
# Find and open file
vim $(fzf)

# Search command history
history | fzf

# Git branch checkout
git checkout $(git branch | fzf)

# Find session
amp session resume $(amp session list | fzf | awk '{print $1}')
```

### With Amplifier

```bash
# Find recipe to run
amp recipes execute $(find recipes -name "*.yaml" | fzf)

# Resume session interactively
amp session list | fzf | xargs amp session resume
```

## bat (Better cat)

Syntax-highlighted file viewer.

### Installation

```bash
# macOS
brew install bat

# Ubuntu/Debian
sudo apt install bat

# Note: on Ubuntu, the command is 'batcat'
```

### Common Uses

```bash
# View file with syntax highlighting
bat config.yaml

# Show line numbers
bat -n script.py

# Show specific lines
bat --line-range 10:20 app.py

# Compare files
bat --diff file1.py file2.py
```

## ripgrep (rg)

Faster grep with smart defaults.

### Installation

```bash
# macOS
brew install ripgrep

# Ubuntu/Debian
sudo apt install ripgrep
```

### Common Uses

```bash
# Search for pattern
rg "TODO" 

# Search specific file type
rg "function" --type js

# Show context
rg "error" -C 3

# Ignore case
rg -i "config"

# List files only
rg -l "import pandas"
```

### vs Amplifier grep

Amplifier's built-in grep uses ripgrep when available, with added features like output limits and smart exclusions. Use `rg` directly when you need full control.

## Shell Aliases

Add to your `~/.bashrc` or `~/.zshrc`:

```bash
# Amplifier shortcuts
alias a='amp'
alias ar='amp run'
alias as='amp session'
alias ab='amp bundle'

# Development
alias py='python3'
alias pip='uv pip'

# Navigation
alias ..='cd ..'
alias ...='cd ../..'

# Git
alias gs='git status'
alias gd='git diff'
alias gc='git commit'
alias gp='git push'
```

## Try It Yourself

### Exercise 1: Install Tools

```bash
# Install the essentials
uv tool install amplifier
brew install jq fzf bat ripgrep  # or apt equivalent
```

### Exercise 2: Explore Session Logs

```bash
# Find latest session
ls -t ~/.amplifier/sessions/ | head -1

# Parse events
cat ~/.amplifier/sessions/[id]/events.jsonl | jq '.event_type' | sort | uniq -c
```

### Exercise 3: Fuzzy Find

```bash
# Install fzf if needed
brew install fzf

# Use it
vim $(fzf)
```

## Next

Set up your development environment:

→ [Shadow Workspace](shadow-workspace.md)
