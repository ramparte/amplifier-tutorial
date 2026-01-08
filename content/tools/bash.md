---
id: tool-bash
type: tools
title: "Bash Tool"
---

# Bash Tool

Execute shell commands to build, test, and manage your project.

## Overview

The bash tool runs shell commands and returns their output. It's how Amplifier:

- Runs tests
- Builds projects
- Manages packages
- Interacts with git
- Runs any CLI tool

## Basic Usage

```
> Run the tests
```

```
[Tool: bash]
command: pytest
```

```
> Install the requests package
```

```
[Tool: bash]
command: pip install requests
```

## Common Commands

### Project Management

```
# Build
> Build the project
[bash] npm run build

# Test
> Run tests with coverage
[bash] pytest --cov=src

# Lint
> Check for linting errors
[bash] ruff check src/
```

### Package Management

```
# Python
> Install dependencies
[bash] pip install -r requirements.txt

# Node.js
> Install packages
[bash] npm install

# Rust
> Build release
[bash] cargo build --release
```

### Git Operations

```
# Status
> Show git status
[bash] git status

# Diff
> Show what changed
[bash] git diff

# Log
> Show recent commits
[bash] git log --oneline -10
```

### File Operations

```
# Find files
> Find all Python files
[bash] find . -name "*.py" -type f

# Count lines
> How many lines of code?
[bash] find . -name "*.py" | xargs wc -l

# Disk usage
> Check disk space
[bash] df -h
```

## Background Processes

For long-running commands (dev servers, watchers):

```
> Start the dev server
```

```
[Tool: bash]
command: npm run dev
run_in_background: true
```

Returns immediately with PID. The server runs in the background.

## Chaining Commands

### Sequential (&&)

Run commands in sequence, stopping on failure:

```
[bash] npm install && npm run build && npm test
```

### Always Run (;)

Run all commands regardless of success:

```
[bash] npm test; npm run lint; npm run typecheck
```

### Pipes (|)

Process output:

```
[bash] cat access.log | grep ERROR | wc -l
```

## Working Directory

Commands run in your current working directory:

```
> What directory am I in?
[bash] pwd
```

To run in a specific directory:

```
[bash] cd /path/to/project && npm test
```

## Environment Variables

```
# Set inline
[bash] DEBUG=true npm test

# Use existing
[bash] echo $HOME

# Export for subsequent commands
[bash] export API_KEY=xxx && node app.js
```

## Safety Features

The bash tool has built-in safety:

### Blocked Commands

These dangerous patterns are blocked:

```
rm -rf /           # Root deletion
sudo rm -rf        # Privileged deletion
:(){ :|:& };:     # Fork bomb
```

### Timeout

Commands timeout after 30 seconds by default. For long operations:

```
> Build might take a while
[bash] timeout 300 npm run build  # 5 minute timeout
```

### Interactive Commands

Interactive commands don't work:

```
# Won't work
[bash] vim file.txt
[bash] python  # interactive shell

# Use non-interactive alternatives
[bash] cat file.txt
[bash] python -c "print('hello')"
```

## Best Practices

### Prefer Specific Tools

Use Amplifier's specialized tools when available:

```
# Instead of:
[bash] cat file.txt

# Use:
[read_file] file.txt
```

```
# Instead of:
[bash] grep -r "pattern" .

# Use:
[grep] pattern: "pattern"
```

### Quote Paths with Spaces

```
[bash] cd "/path/with spaces" && ls
```

### Use Absolute Paths

When in doubt, use absolute paths:

```
[bash] /usr/bin/python3 /full/path/to/script.py
```

## Try It Yourself

### Exercise 1: Explore Your System

```
> What version of Python is installed?
> What's my current directory?
> How much disk space is free?
```

### Exercise 2: Project Commands

```
> List all package.json files in this directory tree
> Show the git log for the last 5 commits
```

### Exercise 3: Combine Commands

```
> Find all TODO comments in Python files and count them
```

## Errors and Troubleshooting

### "Command not found"

The command isn't in PATH:

```
# Check if installed
[bash] which python3

# Use full path
[bash] /usr/bin/python3 script.py
```

### "Permission denied"

Need execute permission:

```
[bash] chmod +x script.sh && ./script.sh
```

### "Timeout"

Command took too long:

```
# Use explicit timeout
[bash] timeout 120 long_running_command
```

### Exit Codes

```
# Check last exit code
[bash] some_command; echo "Exit code: $?"
```

| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | General error |
| 126 | Permission denied |
| 127 | Command not found |
| 128+N | Killed by signal N |
