---
id: key-commands
type: quickstart
title: "Key Commands"
---

# Key Commands

This reference covers the essential commands you'll use every day with Amplifier. Master these three commands and you'll have complete control over your sessions.

## Command Overview

| Command | Purpose | When to Use |
|---------|---------|-------------|
| `amplifier run` | Start a new session | Beginning fresh work |
| `amplifier continue` | Resume the last session | Picking up where you left off |
| `amplifier resume` | Choose from session history | Returning to older work |

## Quick Reference

```bash
# Start fresh
amplifier run

# Continue last session
amplifier continue

# Pick a specific session
amplifier resume
```

---

## amplifier run

The `run` command starts a new Amplifier session in your current directory.

### Basic Usage

```bash
amplifier run
```

This launches an interactive session where you can start working with the AI assistant immediately.

### Common Options

```bash
# Start with a specific prompt
amplifier run "Explain the architecture of this project"

# Use a specific bundle configuration
amplifier run --bundle my-bundle

# Start in a different directory
amplifier run --cwd /path/to/project

# Enable verbose output for debugging
amplifier run --verbose
```

### With Initial Prompt

Pass your first message directly to skip the initial prompt:

```bash
amplifier run "Review the authentication module for security issues"
```

This immediately starts working on your request without waiting for input.

### Working Directory

Amplifier runs in your current directory by default. The working directory determines:

- Which `AGENTS.md` configuration files are loaded
- The scope of file operations
- Git repository context

```bash
# Run in current directory
cd my-project
amplifier run

# Or specify explicitly
amplifier run --cwd ~/projects/my-app
```

### Bundle Selection

Bundles define what tools and agents are available:

```bash
# Use the default foundation bundle
amplifier run

# Use a specific bundle
amplifier run --bundle developer

# Use a custom bundle from your project
amplifier run --bundle .amplifier/bundles/custom.yaml
```

---

## amplifier continue

The `continue` command resumes your most recent session. This is your go-to command when returning to work.

### Basic Usage

```bash
amplifier continue
```

This picks up exactly where you left off, with full conversation history intact.

### How It Works

1. Amplifier finds your most recent session in the current directory
2. Loads the complete conversation history
3. Restores the session state
4. You can continue the conversation seamlessly

### Typical Workflow

```bash
# Morning: Start working
amplifier run "Let's build the user dashboard"

# ... work for a while, then close terminal ...

# Afternoon: Pick up where you left off
amplifier continue
```

### Session Context

When you continue a session:

- All previous messages are available
- File changes made during the session are remembered
- The AI retains context about what you were working on
- You can reference earlier parts of the conversation

### With Additional Prompt

Resume and immediately send a new message:

```bash
amplifier continue "Now let's add the export feature we discussed"
```

### Directory Matters

`continue` looks for sessions started in your current directory:

```bash
cd my-project
amplifier continue  # Continues last session in my-project

cd other-project
amplifier continue  # Continues last session in other-project
```

---

## amplifier resume

The `resume` command lets you choose from your session history. Use this when you need to return to an older session or switch between projects.

### Basic Usage

```bash
amplifier resume
```

This displays an interactive session picker showing recent sessions.

### Session List

The picker shows:

- Session start time
- Working directory
- First message or topic
- Session duration

Navigate with arrow keys and press Enter to select.

### Resume by ID

If you know the session ID, resume directly:

```bash
amplifier resume abc123
```

Session IDs are shown in the session picker and in session logs.

### Filtering Sessions

Find sessions more easily:

```bash
# Show more sessions
amplifier resume --limit 20

# Filter by directory
amplifier resume --cwd ~/projects/api
```

### When to Use Resume vs Continue

| Scenario | Command |
|----------|---------|
| Return to what you were just doing | `continue` |
| Switch to a different project's session | `resume` |
| Find a session from earlier today | `resume` |
| Access a session from a different directory | `resume` |

---

## Slash Commands

During an active session, use slash commands for quick actions.

### /help

Display available commands and usage information:

```
/help
```

Shows:
- All available slash commands
- Brief description of each
- Usage examples

### /compact

Compress the conversation to save context space:

```
/compact
```

This summarizes earlier messages while preserving important context. Use when:

- The conversation is getting long
- You're hitting context limits
- You want to focus on recent work

After compacting:
- Key decisions and context are preserved
- Detailed earlier messages are summarized
- You have more room for new conversation

### /tools

List available tools in the current session:

```
/tools
```

Shows all tools the AI can use, including:
- File operations (read, write, edit)
- Search tools (grep, glob)
- Shell commands (bash)
- Web tools (search, fetch)
- Agent delegation (task)

### Other Useful Commands

| Command | Purpose |
|---------|---------|
| `/status` | Show session status |
| `/clear` | Clear the screen |
| `/exit` | End the session |
| `/undo` | Undo last action |

---

## Keyboard Shortcuts

While in a session, these shortcuts help navigation:

| Shortcut | Action |
|----------|--------|
| `Ctrl+C` | Cancel current operation |
| `Ctrl+D` | End session (EOF) |
| `Up/Down` | Navigate input history |
| `Tab` | Auto-complete file paths |

---

## Command Cheat Sheet

```bash
# === Starting Sessions ===
amplifier run                    # New session
amplifier run "prompt"           # New session with initial prompt
amplifier run --bundle dev       # New session with specific bundle

# === Resuming Sessions ===
amplifier continue               # Resume last session
amplifier continue "prompt"      # Resume with new message
amplifier resume                 # Pick from history
amplifier resume <id>            # Resume specific session

# === In-Session Commands ===
/help                            # Show help
/compact                         # Compress context
/tools                           # List available tools
/exit                            # End session
```

---

## Troubleshooting

### "No session found to continue"

You haven't started a session in this directory yet:

```bash
# Start a new session instead
amplifier run
```

### Session won't resume

If a session is corrupted or won't load:

```bash
# Start fresh
amplifier run

# Or try a different session
amplifier resume
```

### Context getting too long

Use the compact command to summarize:

```
/compact
```

---

## Next Steps

Now that you know the key commands:

- **[Your First Session](first-session.md)** - Put these commands into practice
- **[Slash Commands Deep Dive](slash-commands.md)** - Learn all in-session commands
- **[Session Management](../guides/session-management.md)** - Advanced session workflows
