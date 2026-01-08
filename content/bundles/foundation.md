---
id: bundle-foundation
type: bundles
title: "Foundation Bundle"
---

# Foundation Bundle

The base bundle that provides core capabilities for Amplifier.

## Overview

Foundation is the recommended starting point for all Amplifier configurations. It provides:

- **Core tools** - File operations, bash, search, web
- **Specialist agents** - Debugger, architect, builder, explorer
- **Philosophy** - Implementation and design principles
- **Behaviors** - Git operations, security review, testing

## What's Included

### Tools

| Tool | Purpose |
|------|---------|
| `read_file` | Read files and directories |
| `write_file` | Create or overwrite files |
| `edit_file` | Make targeted edits |
| `bash` | Run shell commands |
| `grep` | Search file contents |
| `glob` | Find files by pattern |
| `web_search` | Search the internet |
| `web_fetch` | Fetch URL content |
| `task` | Spawn sub-agents |
| `todo` | Track tasks |

### Agents

| Agent | Specialty |
|-------|-----------|
| `zen-architect` | Design, architecture, code review |
| `bug-hunter` | Debugging and troubleshooting |
| `modular-builder` | Implementation from specs |
| `explorer` | Codebase reconnaissance |
| `security-guardian` | Security review |
| `test-coverage` | Testing strategy |
| `git-ops` | Git and GitHub operations |
| `file-ops` | Bulk file operations |
| `web-research` | Internet research |
| `integration-specialist` | API and service integration |
| `post-task-cleanup` | Workspace hygiene |
| `session-analyst` | Debug Amplifier sessions |

### Hooks

| Hook | Purpose |
|------|---------|
| `hooks-logging` | Event logging to JSONL |
| `hooks-streaming-ui` | Stream responses |
| `hooks-status-context` | Inject environment info |
| `hooks-todo-reminder` | Task list reminders |

### Philosophy Context

Foundation loads these philosophy documents:

- **IMPLEMENTATION_PHILOSOPHY.md** - Ruthless simplicity
- **MODULAR_DESIGN_PHILOSOPHY.md** - Bricks and studs
- **KERNEL_PHILOSOPHY.md** - Mechanism not policy

## Using Foundation

### As Your Bundle

```bash
# Foundation is often the default
amplifier
```

### Including in Custom Bundles

```yaml
# my-bundle/bundle.yaml
bundle:
  name: my-custom
  version: 1.0.0

includes:
  - bundle: foundation  # Gets all foundation capabilities

# Add your customizations on top
```

## Agent Details

### zen-architect

Design and architecture specialist:

```
> Use zen-architect to review this module's design
```

- Analyzes code structure
- Identifies complexity issues
- Suggests refactoring
- Creates specifications

### bug-hunter

Debugging specialist:

```
> Debug why authentication is failing
```

- Hypothesis-driven debugging
- Traces call chains
- Identifies root causes
- Suggests fixes

### modular-builder

Implementation specialist:

```
> Implement the caching layer from the spec
```

- Builds from specifications
- Follows modular design
- Creates regeneratable code
- Includes tests

### explorer

Codebase reconnaissance:

```
> Map the structure of this project
```

- Breadth-first exploration
- Summarizes architecture
- Identifies key components
- Creates mental maps

### security-guardian

Security review specialist:

```
> Review src/auth.py for vulnerabilities
```

- OWASP Top 10 checks
- Auth/authz review
- Input validation
- Secret detection

### git-ops

Git and GitHub operations:

```
> Commit these changes with a good message
```

- Quality commit messages
- PR creation
- Branch management
- Safety protocols

## Configuration

Foundation can be configured:

```yaml
includes:
  - bundle: foundation
    config:
      # Customize which agents are available
      agents:
        exclude:
          - session-analyst  # If you don't need it
      
      # Customize hooks
      hooks:
        approval:
          require_for:
            - bash
            - write_file
```

## Philosophy in Practice

Foundation embeds these principles:

### Ruthless Simplicity

```
"As simple as possible, but no simpler"
"Every abstraction must justify its existence"
```

### Bricks and Studs

```
"Modules are self-contained bricks"
"Interfaces are stable studs"
"Regenerate any piece independently"
```

### Mechanism Not Policy

```
"Kernel provides capabilities"
"Modules make decisions"
```

## Try It Yourself

### Exercise 1: See What's Available

```bash
amplifier

> What agents and tools do you have?
```

### Exercise 2: Use Specialists

```
> Use zen-architect to review src/main.py
> Use explorer to map the project structure
```

### Exercise 3: Check Philosophy

```
> What are the core implementation principles you follow?
```

## Source

Foundation bundle is part of `amplifier-foundation`:

```
github.com/microsoft/amplifier-foundation
├── bundles/
│   └── foundation/
│       ├── bundle.yaml
│       ├── agents/
│       ├── behaviors/
│       └── context/
```
