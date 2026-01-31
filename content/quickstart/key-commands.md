---
id: key-commands
type: quickstart
title: "Key Commands"
---

# Key Commands

Essential commands for working with Amplifier CLI and managing your AI development sessions.

## Command Overview

| Command | Purpose | When to Use |
|---------|---------|-------------|
| `amplifier run` | Start new session | Beginning a new task or conversation |
| `amplifier continue` | Resume last session | Pick up where you left off |
| `amplifier resume` | Pick specific session | Return to any previous session |
| `amplifier list` | View all sessions | See your session history |
| `amplifier export` | Export session | Save conversation for sharing |

## amplifier run

Start a new Amplifier session with an initial instruction.

### Basic Usage

```bash
# Start with a prompt
amplifier run "Create a REST API for user management"

# Start with empty prompt (interactive mode)
amplifier run
```

### Common Options

```bash
# Specify an agent
amplifier run --agent foundation:explorer "Analyze the codebase structure"

# Run with a specific model
amplifier run --model claude-3-5-sonnet-20241022 "Review code quality"

# Set working directory
amplifier run --dir /path/to/project "Add unit tests"

# Load a skill before starting
amplifier run --skill python-standards "Refactor this module"
```

### Advanced Options

```bash
# Run in non-interactive mode (CI/CD)
amplifier run --non-interactive "Run all tests"

# Set temperature for creativity
amplifier run --temperature 0.7 "Generate creative variable names"

# Enable debug logging
amplifier run --debug "Investigate this bug"

# Use a specific configuration file
amplifier run --config custom-config.yaml "Deploy to production"
```

### Environment Variables

```bash
# Set API key
export ANTHROPIC_API_KEY=your_key_here

# Set custom config location
export AMPLIFIER_CONFIG_PATH=~/.config/amplifier/config.yaml

# Enable verbose logging
export AMPLIFIER_LOG_LEVEL=debug
```

### Examples

```bash
# Web development task
amplifier run "Create a React component for user profile"

# Code review
amplifier run --agent foundation:bug-hunter "Find security issues"

# Documentation
amplifier run "Generate API documentation for all endpoints"

# Testing
amplifier run --agent foundation:test-coverage "Add tests for auth module"
```

## amplifier continue

Resume your most recent Amplifier session, picking up exactly where you left off.

### Basic Usage

```bash
# Continue the last session
amplifier continue

# Continue with a new instruction
amplifier continue "Now add authentication to the API"
```

### When to Use

- **Quick iterations**: You just closed a session and want to continue
- **Morning workflow**: Resume yesterday's work without searching
- **Active development**: Rapidly iterate on the current task

### Options

```bash
# Continue with different model
amplifier continue --model claude-3-5-sonnet-20241022

# Continue with additional context
amplifier continue --skill security-best-practices
```

### Workflow Example

```bash
# Day 1: Start working on a feature
amplifier run "Create user authentication system"
# ... work session ...
# Exit with Ctrl+D

# Day 2: Continue where you left off
amplifier continue "Add password reset functionality"
```

## amplifier resume

Select and resume any previous session from your history.

### Basic Usage

```bash
# Show list of sessions and pick one
amplifier resume

# Resume specific session by ID
amplifier resume abc123def456
```

### Interactive Session Selection

When you run `amplifier resume` without a session ID, you'll see:

```
Recent Sessions:
1. [2 hours ago] "Create REST API for user management"
2. [1 day ago] "Add unit tests for authentication"
3. [2 days ago] "Refactor database models"
4. [1 week ago] "Setup CI/CD pipeline"

Select a session (1-4) or enter session ID:
```

### Advanced Usage

```bash
# Resume with filters
amplifier resume --agent foundation:bug-hunter
amplifier resume --date 2024-01-15
amplifier resume --grep "authentication"

# Resume and immediately add instruction
amplifier resume abc123 "Continue with the OAuth integration"
```

### Use Cases

- **Context switching**: Return to a paused project
- **Code archaeology**: Review past decisions and conversations
- **Learning**: Revisit how you solved similar problems
- **Collaboration**: Resume a session started by a teammate

## amplifier list

View and manage your session history.

### Basic Usage

```bash
# List all sessions
amplifier list

# List with details
amplifier list --verbose

# List recent sessions only
amplifier list --limit 10
```

### Filtering Options

```bash
# Filter by date
amplifier list --since "2024-01-01"
amplifier list --until "2024-01-31"

# Filter by agent
amplifier list --agent foundation:explorer

# Search by content
amplifier list --grep "authentication"

# Filter by status
amplifier list --active
amplifier list --completed
```

## amplifier export

Export session transcripts for sharing, documentation, or archival.

### Basic Usage

```bash
# Export last session
amplifier export

# Export specific session
amplifier export abc123def456

# Export to file
amplifier export abc123 --output session-log.md
```

### Export Formats

```bash
# Markdown (default)
amplifier export --format markdown

# JSON for processing
amplifier export --format json

# HTML for viewing
amplifier export --format html
```

## Slash Commands

Special commands available during an active Amplifier session.

### Help and Information

```
/help                 Show available slash commands
/tools                List available tools and their capabilities
/skills               Show loaded skills
/context              Display current context information
```

### Display Control

```
/compact              Toggle compact mode (less verbose output)
/verbose              Toggle verbose mode (more detailed output)
/clear                Clear the terminal screen
```

### Session Management

```
/status               Show current session status
/save                 Save session checkpoint
/export               Export current session
/quit                 Exit session (Ctrl+D also works)
```

### Context and Memory

```
/forget [topic]       Remove specific context from memory
/summarize            Get summary of conversation so far
/tokens               Show token usage statistics
```

### Advanced Commands

```
/agent [name]         Switch to a different agent
/model [name]         Switch to a different model
/temperature [0-1]    Adjust response creativity
/skill [name]         Load additional skill
```

### Examples

```
# During a session
You: Create a user authentication system
AI: [creates authentication code]

You: /compact
# Output mode changed to compact

You: /tools
# Shows: bash, read_file, write_file, grep, etc.

You: /skill security-best-practices
# Loads security skill for additional guidance
```

## Tips and Best Practices

### Session Management

- **Use descriptive prompts**: Start sessions with clear, specific goals
- **Resume frequently**: Don't hesitate to continue previous sessions
- **Export important sessions**: Save key conversations for documentation

### Command Efficiency

```bash
# Use aliases for common commands
alias amp='amplifier'
alias ampr='amplifier run'
alias ampc='amplifier continue'

# Chain commands
amplifier run "Setup project" && amplifier continue "Add tests"
```

### Working with Multiple Projects

```bash
# Use project-specific scripts
cd ~/project-a && amplifier run "Add feature X"
cd ~/project-b && amplifier run "Fix bug Y"

# Or use --dir flag
amplifier run --dir ~/project-a "Add feature X"
```

### Debugging Issues

```bash
# Enable verbose logging
amplifier run --debug "Investigate this error"

# Check configuration
amplifier config show

# Verify installation
amplifier version
amplifier doctor
```

## Keyboard Shortcuts

| Shortcut | Action |
|----------|--------|
| `Ctrl+D` | Exit session gracefully |
| `Ctrl+C` | Interrupt current operation |
| `Ctrl+L` | Clear screen (same as /clear) |
| `Up/Down` | Navigate command history |
| `Tab` | Auto-complete (if available) |

## Next Steps

### Continue Learning

- **[First Conversation](./first-conversation.md)**: Practice with your first Amplifier session
- **[Agent System](../concepts/agents.md)**: Understand specialized agents
- **[Bundles](../concepts/bundles.md)**: Extend with custom capabilities

### Explore Tools

- **[Task Tool](../tools/task.md)**: Launch specialized sub-agents
- **[Recipe System](../tools/recipes-tool.md)**: Automate multi-step workflows
- **[File Operations](../tools/filesystem.md)**: Master file manipulation

### Advanced Usage

- **[Skills](../concepts/skills.md)**: Load domain expertise
- **[Advanced Topics](../advanced/index.md)**: Custom bundles, recipes, and tools
- **[Concepts](../concepts/index.md)**: Deep understanding of Amplifier architecture

## Quick Reference Card

```bash
# Start new session
amplifier run "Your instruction here"

# Continue last session
amplifier continue

# Pick any session
amplifier resume

# List all sessions
amplifier list

# Get help
amplifier --help
amplifier run --help

# During session
/help        # Show commands
/tools       # List tools
/compact     # Toggle output mode
Ctrl+D       # Exit
```

---

**Need Help?** Run `amplifier --help` for command documentation or visit the [documentation site](https://amplifier.dev) for comprehensive guides.
