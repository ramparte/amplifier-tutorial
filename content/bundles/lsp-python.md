---
id: bundle-lsp-python
type: bundles
title: "LSP Python Bundle"
---

# LSP Python Bundle

Python code intelligence using Pyright language server.

## Overview

The LSP Python bundle adds semantic code navigation:

- **Go to definition** - Jump to where symbols are defined
- **Find references** - Find all usages of a symbol
- **Hover** - Get type info and documentation
- **Call hierarchy** - Trace what calls what

## Installation

```bash
# Add the bundle
amplifier bundle add git+https://github.com/microsoft/amplifier-bundle-lsp-python@main

# Use it
amplifier bundle use lsp-python
```

### Prerequisites

Pyright must be installed:

```bash
# Via npm (recommended)
npm install -g pyright

# Verify
pyright --version
```

## What's Included

### Tools

| Tool | Operations |
|------|------------|
| `LSP` | goToDefinition, findReferences, hover, documentSymbol, workspaceSymbol, incomingCalls, outgoingCalls |

### Agents

| Agent | Purpose |
|-------|---------|
| `python-code-intel` | Complex multi-step code navigation |

## LSP Operations

### goToDefinition

```
> Where is the User class defined?
```

```
[Tool: LSP]
operation: goToDefinition
file_path: src/api/users.py
line: 15
character: 20
```

Returns exact file and line.

### findReferences

```
> Find all usages of authenticate()
```

```
[Tool: LSP]
operation: findReferences
file_path: src/auth.py
line: 25
character: 8
```

Returns all files and locations.

### hover

```
> What type does get_config() return?
```

```
[Tool: LSP]
operation: hover
file_path: src/config.py
line: 10
character: 5
```

Returns type signature and docstring.

### incomingCalls

```
> What calls validate_token()?
```

Shows all functions that call the target.

### outgoingCalls

```
> What does process_request() call?
```

Shows all functions called by the target.

## LSP vs Grep

| Task | Use LSP | Use Grep |
|------|---------|----------|
| Find definition | ✅ Exact | ❌ Multiple matches |
| Find all callers | ✅ Semantic | ❌ Text matches |
| Get type info | ✅ Full types | ❌ Not possible |
| Search text | ❌ Wrong tool | ✅ Fast |

**Rule**: LSP for understanding, grep for searching.

## Python-Specific Features

### Type Inference

Pyright infers types even without annotations:

```python
def get_user():
    return {"name": "Alice", "age": 30}

user = get_user()
# [hover] → user: dict[str, str | int] (inferred)
```

### Virtual Environment Support

Pyright detects your environment:

```
pyproject.toml → Python version, dependencies
.venv/ → Virtual environment
requirements.txt → Dependencies
```

### Stub Packages

For better type info, install stubs:

```bash
pip install types-requests types-redis
```

## Code Intel Agent

For complex navigation tasks:

```
> Trace the authentication flow from login to database
```

The `python-code-intel` agent:
- Performs multi-step navigation
- Traces inheritance chains
- Maps module dependencies
- Understands type flows

## Common Workflows

### Understand a Function

```
> I need to understand how authenticate() works
```

1. `hover` - Get signature
2. `outgoingCalls` - What it calls
3. `incomingCalls` - What calls it

### Prepare for Refactoring

```
> I want to rename UserService
```

1. `findReferences` - All usages
2. Know exactly what to change

### Debug Type Issues

```
> Why is this type error happening?
```

1. `hover` - Check actual types
2. Trace through call chain

## Try It Yourself

### Exercise 1: Find Definition

```
> Where is [class in your project] defined?
```

### Exercise 2: Find Usages

```
> Find all usages of [function name]
```

### Exercise 3: Get Type Info

```
> What type does [function] return?
```

## Troubleshooting

### "No results"

- Check Pyright is installed: `pyright --version`
- Ensure cursor is on the symbol name
- Wait for initial indexing

### Slow First Query

Language server indexes on startup. Subsequent queries are fast.

### Missing Type Info

- Install stub packages
- Add type annotations
- Check `pyproject.toml` settings

## Configuration

```yaml
# In bundle or settings
tools:
  - module: tool-lsp
    config:
      python:
        venv_path: ./.venv
        python_version: "3.12"
```

## Source

```
github.com/microsoft/amplifier-bundle-lsp-python
├── bundle.yaml
├── context/
│   └── python-lsp.md
└── agents/
    └── python-code-intel.yaml
```
