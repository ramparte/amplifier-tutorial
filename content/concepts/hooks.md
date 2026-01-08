---
id: hooks
type: concepts
title: "Hooks"
---

# Hooks

Hooks observe and optionally modify the Amplifier lifecycle. They're how you add logging, require approvals, redact sensitive data, and more.

## What is a Hook?

A hook is code that runs at specific points in the agent lifecycle:

```
Session Start
    ↓
Turn Start → Provider Request → Provider Response
    ↓              ↓
Tool Call → Tool Result
    ↓
Turn End
    ↓
Session End
```

Hooks can:
- **Observe** - Watch what happens (logging)
- **Enrich** - Add data to events (inject status)
- **Modify** - Change data (redaction)
- **Cancel** - Block an action (approval required)

## Available Hooks

| Hook | Purpose | Action |
|------|---------|--------|
| `hooks-logging` | Log events to JSONL | Observe |
| `hooks-streaming-ui` | Stream to terminal | Observe |
| `hooks-status-context` | Inject environment info | Enrich |
| `hooks-todo-reminder` | Remind about tasks | Enrich |
| `hooks-redaction` | Remove sensitive data | Modify |
| `hooks-approval` | Require human approval | Cancel |

## Configuring Hooks

In your bundle or settings:

```yaml
hooks:
  # Basic logging
  - module: hooks-logging
    config:
      output_dir: ~/.amplifier/logs
      
  # Stream responses to terminal
  - module: hooks-streaming-ui
  
  # Inject git status, date, etc.
  - module: hooks-status-context
  
  # Redact API keys from logs
  - module: hooks-redaction
    config:
      patterns:
        - "sk-[a-zA-Z0-9]+"
        - "api_key=[^&]+"
```

## Hook: Logging

Records all events to a JSONL file:

```yaml
hooks:
  - module: hooks-logging
    config:
      output_dir: ~/.amplifier/sessions
```

Output:
```jsonl
{"event_type":"session:start","timestamp":"2026-01-07T10:00:00Z",...}
{"event_type":"turn:start","timestamp":"2026-01-07T10:00:01Z",...}
{"event_type":"tool:call","tool":"read_file","timestamp":"..."}
```

This is how Amplifier maintains transparency - everything is logged.

## Hook: Approval Gates

Require human approval for dangerous operations:

```yaml
hooks:
  - module: hooks-approval
    config:
      require_approval_for:
        - bash           # Shell commands
        - write_file     # File creation
        - edit_file      # File modification
```

When triggered:

```
[Tool: bash] Running: rm -rf temp/
⚠️  This action requires approval.

Approve? [y/n]: _
```

### Approval Patterns

```yaml
# Approve specific commands
require_approval_for:
  - bash:rm        # Only rm commands
  - bash:sudo      # Only sudo commands
  
# Approve by path
require_approval_for:
  - write_file:src/   # Writing to src/
  - edit_file:*.py    # Editing Python files
```

## Hook: Redaction

Remove sensitive data from logs:

```yaml
hooks:
  - module: hooks-redaction
    config:
      patterns:
        # API keys
        - "sk-[a-zA-Z0-9]{20,}"
        - "api_key=[^&\\s]+"
        
        # Passwords in URLs
        - "://[^:]+:[^@]+@"
        
        # Custom secrets
        - "SECRET_[A-Z_]+=[^\\s]+"
```

Before:
```
Connecting to postgres://admin:secretpass@db.example.com
```

After (in logs):
```
Connecting to postgres://[REDACTED]@db.example.com
```

## Hook: Status Context

Inject environment information into the AI's context:

```yaml
hooks:
  - module: hooks-status-context
    config:
      include:
        - git_status
        - working_dir
        - date_time
        - platform
```

The AI then knows:
- Current git branch and status
- Working directory
- Current date/time
- Operating system

## Hook Events

### Session Events

| Event | When | Use Case |
|-------|------|----------|
| `session:start` | Session begins | Initialize state |
| `session:end` | Session ends | Cleanup, summary |

### Turn Events

| Event | When | Use Case |
|-------|------|----------|
| `turn:start` | User message received | Log input |
| `turn:end` | Response complete | Log output |

### Provider Events

| Event | When | Use Case |
|-------|------|----------|
| `provider:request` | Before API call | Log/modify request |
| `provider:response` | After API response | Log/modify response |

### Tool Events

| Event | When | Use Case |
|-------|------|----------|
| `tool:call` | Tool invoked | Approve, log |
| `tool:result` | Tool returns | Log result |

## Creating Custom Hooks

```python
# hooks/my_hook.py
from amplifier_core import Hook, HookResult

class MyHook(Hook):
    name = "my-custom-hook"
    
    async def on_tool_call(self, event):
        """Called before each tool execution."""
        tool_name = event.tool_name
        
        # Log it
        print(f"Tool called: {tool_name}")
        
        # Optionally block it
        if tool_name == "dangerous_tool":
            return HookResult.cancel("This tool is disabled")
        
        # Allow it
        return HookResult.continue_()
    
    async def on_session_end(self, event):
        """Called when session ends."""
        # Generate summary, cleanup, etc.
        print(f"Session ended: {event.session_id}")
```

Register in bundle:

```yaml
hooks:
  - module: my-custom-hook
    source: ./hooks/my_hook.py
```

## Hook Ordering

Hooks run in order. Earlier hooks can affect later ones:

```yaml
hooks:
  # 1. First, check approval
  - module: hooks-approval
  
  # 2. Then log (only if approved)
  - module: hooks-logging
  
  # 3. Then redact logged content
  - module: hooks-redaction
```

## Try It Yourself

### Exercise 1: View Your Logs

```bash
# Find recent session
ls -la ~/.amplifier/sessions/

# View events
cat ~/.amplifier/sessions/[session-id]/events.jsonl | head -10
```

### Exercise 2: Add Approval Gates

Create a bundle with approval:

```yaml
# approval-bundle/bundle.yaml
bundle:
  name: careful-mode
  version: 1.0.0

includes:
  - bundle: foundation

hooks:
  - module: hooks-approval
    config:
      require_approval_for:
        - bash
        - write_file
```

Use it:
```bash
amplifier --bundle ./approval-bundle
> Create a file called test.txt
# You'll be prompted to approve
```

### Exercise 3: Check What Hooks Are Active

```bash
amplifier

> What hooks are currently active? What do they do?
```

## Key Takeaways

1. **Hooks observe lifecycle** - Every event can be watched
2. **Four actions** - Observe, enrich, modify, cancel
3. **Configurable per-bundle** - Different contexts, different hooks
4. **Transparency** - Logging hooks make everything visible

## Summary

You've now learned all the core concepts:

| Concept | Role |
|---------|------|
| **Bundles** | Package and configure everything |
| **Modules** | Building blocks (tools, providers) |
| **Agents** | Specialized AI configurations |
| **Recipes** | Multi-step workflows |
| **Skills** | Domain knowledge packages |
| **Hooks** | Lifecycle observation and control |

Ready to explore specific capabilities? Check the [Tools Reference](../tools/index.md) or [Developer Setup](../dev-setup/index.md).
