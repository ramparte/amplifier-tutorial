---
id: tool-task
type: tools
title: "Task Tool (Sub-Agents)"
---

# Task Tool

Spawn specialized sub-agents to handle complex tasks.

## Overview

The task tool lets the main AI delegate work to specialist agents:

```
Main AI: "I'll delegate this debugging to bug-hunter"

[Tool: task]
agent: foundation:bug-hunter
instruction: "Debug the authentication failure in src/auth.py"

[Bug-hunter runs, investigates, returns findings]

Main AI: "The bug-hunter found the issue..."
```

## When to Use

The task tool is used automatically when:

- **Complex debugging** → `bug-hunter` agent
- **Architecture questions** → `zen-architect` agent
- **Code implementation** → `modular-builder` agent
- **Codebase exploration** → `explorer` agent
- **Security review** → `security-guardian` agent
- **File operations** → `file-ops` agent
- **Git operations** → `git-ops` agent

## How It Works

```
┌─────────────────────────────────────┐
│          Main Session               │
│                                     │
│  You: "Debug this auth issue"       │
│                                     │
│  Main AI: [spawns sub-agent]        │
│      │                              │
│      ▼                              │
│  ┌─────────────────────────────┐    │
│  │     bug-hunter agent        │    │
│  │  - Specialized instructions │    │
│  │  - Focused tool access      │    │
│  │  - Domain context loaded    │    │
│  │                             │    │
│  │  [Investigates...]          │    │
│  │  [Returns findings]         │    │
│  └─────────────────────────────┘    │
│      │                              │
│      ▼                              │
│  Main AI: "Found the issue..."      │
└─────────────────────────────────────┘
```

Key points:
- Sub-agents are **isolated sessions**
- They receive **focused instructions**
- They have **specialized context**
- They **return results** to the main session

## Available Agents

### Foundation Agents

| Agent | Specialty | Triggers |
|-------|-----------|----------|
| `foundation:zen-architect` | Design, planning | "design", "architect", "review" |
| `foundation:bug-hunter` | Debugging | "debug", "error", "fix bug" |
| `foundation:modular-builder` | Implementation | "implement", "build", "create" |
| `foundation:explorer` | Reconnaissance | "explore", "find", "understand" |
| `foundation:security-guardian` | Security | "security", "vulnerability" |
| `foundation:test-coverage` | Testing | "test", "coverage" |
| `foundation:git-ops` | Git operations | "commit", "PR", "branch" |
| `foundation:file-ops` | File operations | Bulk file changes |
| `foundation:web-research` | Web research | "search", "find online" |

### Design Intelligence Agents

| Agent | Specialty |
|-------|-----------|
| `design-intelligence:art-director` | Visual strategy |
| `design-intelligence:component-designer` | UI components |
| `design-intelligence:layout-architect` | Page structure |

## Explicit Delegation

You can explicitly request an agent:

```
> Use the security-guardian to review src/auth.py
```

```
> Ask zen-architect to design a caching layer
```

```
> Have bug-hunter investigate why tests fail
```

## Task Tool Parameters

```yaml
agent: foundation:bug-hunter      # Which agent
instruction: "Debug auth.py"       # What to do
session_id: "abc123"               # Optional: resume session
```

### Resuming Sessions

Sub-agent sessions can be resumed:

```
[Tool: task]
agent: foundation:explorer
session_id: "previous-session-id"
instruction: "Continue exploring the database module"
```

## What Agents See

Sub-agents receive:

1. **Instruction** - The specific task
2. **Context** - Relevant files, previous findings
3. **Tools** - Subset appropriate to their specialty
4. **Domain knowledge** - Specialized system prompt

They do NOT see:
- Full conversation history
- Unrelated context
- All tools (just relevant ones)

## Best Practices

### Be Specific in Instructions

```
# Vague
instruction: "Fix the bug"

# Specific
instruction: |
  Debug the authentication failure in src/auth.py.
  Error: "Invalid token" when token is valid.
  Check: Token generation, validation, expiry.
```

### Provide Context

```
instruction: |
  Review this module for security issues:
  
  File: src/api/users.py
  Context: Handles user registration and login
  Concern: SQL injection and auth bypass
```

### Use the Right Agent

| Task | Best Agent |
|------|------------|
| "Why is this slow?" | `bug-hunter` |
| "Design a solution" | `zen-architect` |
| "Write the code" | `modular-builder` |
| "What does this codebase do?" | `explorer` |
| "Check for vulnerabilities" | `security-guardian` |

## Try It Yourself

### Exercise 1: Watch Delegation

```
> Debug why this function returns None when it shouldn't:
def get_user(id):
    user = db.query(User).filter(id=id).first()
    return user
```

Watch the output - you'll see it delegate to `bug-hunter`.

### Exercise 2: Explicit Delegation

```
> Use the explorer agent to map the structure of this project
```

### Exercise 3: Chain Agents

```
> First use zen-architect to design a caching solution,
> then use modular-builder to implement it
```

## Agent Reports

When an agent completes, it returns a structured report:

```
[bug-hunter] Investigation complete:

Finding: The query filter syntax is incorrect.
- `filter(id=id)` should be `filter(User.id == id)`

Root cause: SQLAlchemy filter() expects comparison expressions.

Fix:
```python
user = db.query(User).filter(User.id == id).first()
```

Confidence: High
```

## When NOT to Use Task

For simple operations, the main AI handles directly:

```
# Simple file read - main AI does it
> Show me config.yaml

# Simple command - main AI does it  
> Run pytest

# Simple search - main AI does it
> Find all TODO comments
```

Task delegation is for **complex, specialized work**.
