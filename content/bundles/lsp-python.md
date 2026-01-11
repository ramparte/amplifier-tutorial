---
id: lsp-python
type: bundles
title: "LSP Python Bundle"
---

# LSP Python Bundle

## Overview

The LSP Python bundle provides semantic code intelligence for Python projects using the Language Server Protocol (LSP) with Pyright as the backend. Unlike text-based search tools like grep, LSP understands your code semantically—it knows the difference between a function definition, a function call, and a comment mentioning that function.

This bundle transforms how you navigate and understand Python codebases by providing:

- **Precise symbol resolution** - Find where functions, classes, and variables are actually defined
- **Type inference** - Get inferred types even in codebases without type hints
- **Call hierarchy tracing** - Understand what calls what and map dependencies
- **Semantic references** - Find all usages of a symbol (not just text matches)

## What's Included

### LSP Tool (Pyright)

The bundle configures the LSP tool with Pyright, a fast static type checker for Python. Available operations:

| Operation | Description |
|-----------|-------------|
| `goToDefinition` | Jump to where a symbol is defined |
| `findReferences` | Find all usages of a symbol |
| `hover` | Get type information and documentation |
| `documentSymbol` | List all symbols in a file |
| `workspaceSymbol` | Search for symbols across the workspace |
| `goToImplementation` | Find implementations of abstract methods |
| `prepareCallHierarchy` | Prepare for call hierarchy queries |
| `incomingCalls` | Find what calls a function |
| `outgoingCalls` | Find what a function calls |

### python-code-intel Agent

A specialized agent for complex, multi-step Python code navigation tasks. While the LSP tool handles single operations, this agent orchestrates multiple LSP calls to answer higher-level questions like:

- "Trace the inheritance chain for this class"
- "Map all dependencies of this module"
- "Show me the complete call graph for this function"

## When to Use

### Use LSP Python When You Need

**Semantic Understanding Over Text Matching**

```
# Finding where authenticate() is defined
# grep: matches comments, strings, and actual definitions
# LSP goToDefinition: finds the exact definition location
```

**Type Information**

```python
# What type does get_connection() return?
# Even without type hints, Pyright infers types from implementation
def get_connection():
    return DatabaseConnection(host="localhost")

# LSP hover reveals: () -> DatabaseConnection
```

**Call Hierarchy Analysis**

```
# Who calls process_payment()?
# LSP incomingCalls gives you the complete call graph
# grep would match "process_payment" in comments and strings too
```

**Refactoring Preparation**

```
# Before renaming a class, find ALL usages
# LSP findReferences catches:
# - Direct instantiations
# - Type hints
# - Inheritance relationships
# - Import statements
```

### LSP vs Grep: Decision Guide

| Task | Use LSP | Use Grep |
|------|---------|----------|
| Find function definition | goToDefinition | Pattern matching |
| Find all callers | incomingCalls | Text search |
| Get type signature | hover | Not possible |
| Search across many files | Limited | Faster |
| Find text in comments | Not possible | Pattern matching |
| Understand inheritance | goToImplementation | Manual tracing |

**Rule of thumb:** Use LSP for understanding code relationships. Use grep for finding text patterns.

## Configuration

The bundle is pre-configured for Python. To use it, ensure your workspace has Python files and Pyright can resolve your imports.

### Requirements

- Python project with standard structure
- Virtual environment recommended (helps Pyright resolve imports)
- `pyproject.toml` or `pyrightconfig.json` for custom configuration (optional)

### Pyright Configuration

For complex projects, create a `pyrightconfig.json`:

```json
{
  "include": ["src"],
  "exclude": ["**/node_modules", "**/__pycache__"],
  "venvPath": ".",
  "venv": ".venv",
  "pythonVersion": "3.11"
}
```

## Try It Yourself

### Example 1: Find a Definition

Navigate to where a function is defined:

```
User: Where is the authenticate function defined?

Amplifier uses LSP goToDefinition to find:
  src/auth/handlers.py:45 - def authenticate(credentials: dict) -> User
```

### Example 2: Trace Callers

Find all code that calls a specific function:

```
User: What calls the validate_token function?

Amplifier uses LSP incomingCalls to map:
  - src/middleware/auth.py:23 - check_auth()
  - src/api/routes.py:89 - protected_endpoint()
  - src/websocket/handlers.py:34 - ws_authenticate()
```

### Example 3: Get Type Information

Understand types even without annotations:

```
User: What type does create_session return?

Amplifier uses LSP hover:
  def create_session(user_id: str) -> Session
  
  Returns a Session object containing authentication
  state and expiration information.
```

### Example 4: Map Module Dependencies

Use the python-code-intel agent for complex analysis:

```
User: Map all the dependencies of the payment module

Agent traces imports and call relationships:
  payment/
  ├── processor.py
  │   ├── imports: stripe, logging, .models
  │   └── calls: validate_card(), create_charge()
  ├── models.py
  │   └── imports: dataclasses, typing
  └── validators.py
      ├── imports: re, .exceptions
      └── called by: processor.validate_card()
```

### Example 5: Understand Inheritance

Trace class hierarchies:

```
User: Show me the inheritance chain for DatabaseAdapter

LSP reveals:
  DatabaseAdapter
    └── inherits from: BaseAdapter (src/core/adapters.py:12)
        └── inherits from: ABC (abc module)
  
  Implementations:
    - PostgresAdapter (src/adapters/postgres.py:8)
    - SQLiteAdapter (src/adapters/sqlite.py:8)
    - MockAdapter (tests/mocks.py:15)
```

### Example 6: Pre-Refactoring Analysis

Before renaming or moving code:

```
User: I want to rename UserService to AuthService. Show me all usages.

LSP findReferences locates:
  Definition:
    src/services/user.py:10 - class UserService

  Usages (23 total):
    src/api/routes.py:5 - from services.user import UserService
    src/api/routes.py:34 - service = UserService()
    src/api/routes.py:78 - UserService.get_by_id(user_id)
    src/tests/test_user.py:12 - class TestUserService
    ... (19 more)
```

## Common Patterns

### Debugging Type Errors

When Pyright reports a type error, use hover to understand what types are inferred:

```python
# Error: Argument of type "str | None" cannot be assigned to "str"
result = process(get_value())  # What's get_value returning?

# Use LSP hover on get_value() to see:
# def get_value() -> str | None
```

### Understanding Legacy Code

When diving into unfamiliar code:

1. Use `documentSymbol` to see the structure of a file
2. Use `goToDefinition` to trace imports and dependencies
3. Use `incomingCalls` to understand how code is used
4. Use `hover` to understand types without reading implementation

### Verifying Refactoring Safety

Before making changes:

1. Use `findReferences` to locate all usages
2. Use `incomingCalls` to understand dependencies
3. Use `goToImplementation` to find all implementations of an interface

## Limitations

- **Startup time:** Pyright needs to index your workspace on first use
- **Dynamic code:** Highly dynamic Python (heavy use of `getattr`, metaclasses) may not be fully understood
- **External libraries:** Type stubs needed for best results with third-party packages
- **Large workspaces:** Very large codebases may have slower response times

## Related Bundles

- **lsp** - Base LSP bundle (language-agnostic)
- **foundation** - Core development tools including grep for text search

## Summary

The LSP Python bundle brings IDE-level code intelligence to your Amplifier workflow. Use it when you need to truly understand code relationships—not just find text patterns. The semantic understanding it provides is invaluable for navigating complex codebases, preparing refactoring, and debugging type issues.

For simple text searches, grep remains faster. But when you need to answer "what calls this?" or "what type is this?", LSP Python is the right tool.
