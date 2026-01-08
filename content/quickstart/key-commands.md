---
id: key-commands
type: quickstart
title: "Key Commands"
---

# Key Commands

Master these commands to work efficiently with Amplifier.

## CLI Commands

### Starting Sessions

```bash
# Interactive chat mode
amplifier

# Single command (no session)
amplifier run "your prompt here"

# Resume last session
amplifier continue

# Resume specific session
amplifier session resume [session-id]
```

### Managing Sessions

```bash
# List all sessions
amplifier session list

# Show session details
amplifier session show [session-id]

# Delete a session
amplifier session delete [session-id]
```

### Provider Management

```bash
# List configured providers
amplifier provider list

# Switch active provider
amplifier provider use anthropic

# Add a new provider
amplifier provider add openai --api-key sk-...

# Remove a provider
amplifier provider remove openai
```

### Bundle Management

```bash
# List installed bundles
amplifier bundle list

# Add a bundle
amplifier bundle add git+https://github.com/microsoft/amplifier-bundle-recipes@main

# Use a specific bundle
amplifier bundle use recipes

# Remove a bundle
amplifier bundle remove recipes
```

### Configuration

```bash
# Show current config
amplifier config show

# Edit config in $EDITOR
amplifier config edit

# Set a specific value
amplifier config set default_provider anthropic
```

## In-Session Commands

These work while you're in an interactive session (after running `amplifier`):

| Command | Description |
|---------|-------------|
| `/help` | Show all available commands |
| `/tools` | List loaded tools and descriptions |
| `/agents` | List available agents |
| `/clear` | Clear conversation history |
| `/exit` | End the session |
| `/model` | Show/change the current model |
| `/compact` | Reduce context by summarizing |

### Examples

```
amplifier> /tools

Available Tools:
  read_file     - Read file contents
  write_file    - Write content to a file
  edit_file     - Make targeted edits
  bash          - Execute shell commands
  grep          - Search file contents
  glob          - Find files by pattern
  web_search    - Search the web
  web_fetch     - Fetch URL content
  task          - Spawn sub-agents
  todo          - Manage task lists
```

```
amplifier> /agents

Available Agents:
  foundation:zen-architect     - Architecture and design
  foundation:bug-hunter        - Debugging specialist
  foundation:modular-builder   - Implementation
  foundation:explorer          - Codebase reconnaissance
  foundation:security-guardian - Security review
  ...
```

## Keyboard Shortcuts

While in interactive mode:

| Shortcut | Action |
|----------|--------|
| `Ctrl+C` | Cancel current operation |
| `Ctrl+D` | Exit session |
| `↑` / `↓` | Navigate history |
| `Ctrl+R` | Search history |

## Power User Patterns

### Piping Input

```bash
# Analyze a file
cat complex_module.py | amplifier run "Explain this code"

# Process command output
git diff | amplifier run "Summarize these changes for a commit message"

# Analyze logs
tail -100 error.log | amplifier run "What's causing these errors?"
```

### Chaining Commands

```bash
# Run multiple related tasks
amplifier run "Create tests for src/auth.py" && \
amplifier run "Run the tests and fix any failures"
```

### Using with Git

```bash
# Pre-commit review
git diff --staged | amplifier run "Review these changes before commit"

# Generate commit message
git diff --staged | amplifier run "Generate a commit message for these changes"
```

### Quick Lookups

```bash
# Documentation queries
amplifier run "How do I use async/await in Python?"

# Code generation
amplifier run "Write a bash one-liner to find all files over 100MB"

# Explanation
amplifier run "What does this regex do: ^(?=.*[A-Z])(?=.*[0-9]).{8,}$"
```

## Environment Variables

```bash
# Override provider for one command
AMPLIFIER_PROVIDER=openai amplifier run "Hello"

# Set log level
AMPLIFIER_LOG_LEVEL=debug amplifier run "Debug this"

# Custom config location
AMPLIFIER_CONFIG=~/my-config.yaml amplifier
```

## Quick Reference Card

```
┌─────────────────────────────────────────────────────────┐
│                 AMPLIFIER QUICK REFERENCE               │
├─────────────────────────────────────────────────────────┤
│ START        amplifier              Interactive mode    │
│              amplifier run "..."    Single command      │
│              amplifier continue     Resume last         │
├─────────────────────────────────────────────────────────┤
│ SESSIONS     session list           Show all            │
│              session resume [id]    Resume specific     │
├─────────────────────────────────────────────────────────┤
│ PROVIDERS    provider list          Show configured     │
│              provider use [name]    Switch provider     │
├─────────────────────────────────────────────────────────┤
│ BUNDLES      bundle list            Show installed      │
│              bundle add [url]       Install bundle      │
│              bundle use [name]      Activate bundle     │
├─────────────────────────────────────────────────────────┤
│ IN-SESSION   /help                  Show commands       │
│              /tools                 List tools          │
│              /agents                List agents         │
│              /exit                  End session         │
└─────────────────────────────────────────────────────────┘
```

## Next

Now let's add your first bundle to unlock new capabilities.

→ [Your First Bundle](first-bundle.md)
