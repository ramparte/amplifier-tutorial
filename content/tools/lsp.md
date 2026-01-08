---
id: tool-lsp
type: tools
title: "LSP Tool (Code Intelligence)"
---

# LSP Tool

Navigate code semantically with Language Server Protocol.

## Overview

The LSP tool provides IDE-like code intelligence:

| Operation | Purpose |
|-----------|---------|
| `goToDefinition` | Find where symbol is defined |
| `findReferences` | Find all usages of symbol |
| `hover` | Get type info and docs |
| `incomingCalls` | What calls this function? |
| `outgoingCalls` | What does this function call? |

## LSP vs Grep

| Task | LSP | Grep |
|------|-----|------|
| Find function definition | Exact location | May find multiple |
| Find all callers | Semantic references | May include strings/comments |
| Get type signature | Full type info | Not possible |
| Understand inheritance | Traces hierarchy | Text matches only |

**Rule**: Use LSP for code understanding, grep for text search.

## Requirements

LSP requires language-specific bundles:

```bash
# Python (Pyright)
amplifier bundle add git+https://github.com/microsoft/amplifier-bundle-lsp-python@main

# TypeScript (coming)
amplifier bundle add git+https://github.com/robotdad/amplifier-bundle-lsp-typescript@main
```

## Operations

### goToDefinition

Find where a symbol is defined:

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

Returns: `src/models/user.py:10` - exact location

### findReferences

Find all usages of a symbol:

```
> Find all usages of the authenticate function
```

```
[Tool: LSP]
operation: findReferences
file_path: src/auth.py
line: 25
character: 8
```

Returns: List of all files and locations using that symbol.

### hover

Get type information and documentation:

```
> What type does get_user() return?
```

```
[Tool: LSP]
operation: hover
file_path: src/api/users.py
line: 30
character: 15
```

Returns:
```
def get_user(id: int) -> Optional[User]

Get a user by ID.

Returns:
    User object if found, None otherwise.
```

### incomingCalls

Find what calls a function:

```
> What calls the validate_token function?
```

```
[Tool: LSP]
operation: incomingCalls
file_path: src/auth.py
line: 50
character: 5
```

Returns: Call hierarchy showing all callers.

### outgoingCalls

Find what a function calls:

```
> What does process_request call internally?
```

```
[Tool: LSP]
operation: outgoingCalls
file_path: src/api/handler.py
line: 100
character: 5
```

Returns: All functions called by `process_request`.

### workspaceSymbol

Search for symbols across the project:

```
[Tool: LSP]
operation: workspaceSymbol
query: "User"
```

Returns: All symbols matching "User" (classes, functions, variables).

### documentSymbol

Get all symbols in a file:

```
[Tool: LSP]
operation: documentSymbol
file_path: src/models/user.py
```

Returns: Classes, methods, functions in that file.

## Common Workflows

### Understand a Function

```
> I need to understand how authenticate() works
```

1. `hover` - Get signature and docs
2. `outgoingCalls` - See what it calls
3. `incomingCalls` - See what calls it

### Refactoring

```
> I want to rename the process_data function
```

1. `findReferences` - Find all usages
2. Know exactly what needs to change

### Trace a Bug

```
> The error originates from validate_input
```

1. `goToDefinition` - Find the function
2. `incomingCalls` - See the call chain
3. `hover` - Check types at each step

## Line/Character Numbers

LSP uses 1-based line and character numbers:

```python
# Line 1
def hello():     # 'hello' starts at character 5
    return "hi"  # Line 3
```

## Try It Yourself

### Exercise 1: Find Definition

```
> Where is [some class in your project] defined?
```

### Exercise 2: Find Usages

```
> Find all usages of [some function] in this project
```

### Exercise 3: Trace Calls

```
> What functions call [main function]?
> What does [main function] call?
```

## Troubleshooting

### "No results"

- Ensure LSP bundle is installed
- Check cursor is on the symbol name
- Language server may need time to index

### "Server not started"

```bash
# Check language server is installed
pyright --version  # For Python
```

### Slow First Query

The language server indexes on first use. Subsequent queries are faster.

## Python-Specific

With `lsp-python` bundle:

- Full Pyright type inference
- Works even without type annotations
- Understands virtual environments
- Respects `pyproject.toml`

```
> What type is this variable?
```

Even if not annotated, Pyright can often infer:

```python
x = get_config()  # No annotation
# [hover] → x: ConfigDict (inferred)
```
