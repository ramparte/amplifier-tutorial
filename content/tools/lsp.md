---
id: tool-lsp
type: tools
title: "LSP Tool"
---

# LSP Tool (Code Intelligence)

The LSP tool provides semantic code understanding through the Language Server Protocol. Unlike text-based search tools, LSP understands your code's actual structure—types, references, call hierarchies, and more.

## What is LSP?

The Language Server Protocol (LSP) is a standardized protocol for providing programming language features like:

- Go to definition
- Find all references
- Hover information (types, documentation)
- Call hierarchy analysis
- Symbol navigation

Amplifier integrates with language servers to give you precise, semantic code intelligence. Instead of matching text patterns, LSP understands what your code actually means.

### Why Semantic Understanding Matters

Consider searching for a function called `process`:

- **Text search (grep):** Finds every occurrence of "process" including comments, strings, variable names, and unrelated functions
- **LSP:** Finds the exact function definition and its actual callers—nothing more, nothing less

This precision becomes critical in large codebases where text matches produce hundreds of false positives.

## LSP vs Grep: When to Use Each

| Task | Use LSP | Use Grep |
|------|---------|----------|
| Find all callers of a function | `incomingCalls` - semantic, precise | May match strings, comments, false positives |
| Find where a symbol is defined | `goToDefinition` - exact location | Multiple matches, manual filtering needed |
| Get type info or signature | `hover` - full type data | **Not possible** with text search |
| Find text pattern anywhere | Use grep instead | Faster for bulk text search |
| Search across many files | Use grep instead | Better for pattern matching |

### The Rule of Thumb

- **LSP:** Semantic code understanding (types, references, call chains)
- **Grep:** Text pattern matching (fast, broad searches)

## Available Operations

The LSP tool supports these operations:

### goToDefinition

Jump to where a symbol is defined. Works for functions, classes, variables, imports, and more.

```
Operation: goToDefinition
File: src/auth/session.py
Line: 45
Character: 12

Result: src/auth/base.py:23 - class SessionManager
```

**Use when:** You see a function call and want to see its implementation.

### findReferences

Find all locations where a symbol is used throughout the codebase.

```
Operation: findReferences
File: src/models/user.py
Line: 15
Character: 7

Result:
  - src/api/routes.py:34
  - src/services/auth.py:78
  - src/utils/validation.py:12
  - tests/test_user.py:45
```

**Use when:** You're refactoring and need to know everywhere a symbol is used.

### hover

Get type information, documentation, and signatures for any symbol.

```
Operation: hover
File: src/database/connection.py
Line: 28
Character: 15

Result:
  def get_connection(
      pool_name: str,
      timeout: float = 30.0
  ) -> AsyncConnection
  
  "Retrieve a connection from the specified pool."
```

**Use when:** You need to understand a function's signature or a variable's type.

### documentSymbol

List all symbols defined in a single file—functions, classes, variables, constants.

```
Operation: documentSymbol
File: src/models/user.py

Result:
  - class User (line 10)
  - def __init__ (line 15)
  - def validate (line 28)
  - def to_dict (line 45)
  - USER_ROLES: list (line 5)
```

**Use when:** You want an overview of a file's structure.

### workspaceSymbol

Search for symbols across the entire workspace by name.

```
Operation: workspaceSymbol
Query: "Session"

Result:
  - class Session (src/auth/session.py:12)
  - class SessionManager (src/auth/manager.py:8)
  - def create_session (src/api/auth.py:34)
  - SESSION_TIMEOUT (src/config.py:15)
```

**Use when:** You know a symbol's name but not its location.

### goToImplementation

Find concrete implementations of an interface or abstract method.

```
Operation: goToImplementation
File: src/providers/base.py
Line: 12
Character: 8

Result:
  - src/providers/openai.py:15 - class OpenAIProvider
  - src/providers/anthropic.py:18 - class AnthropicProvider
```

**Use when:** You have an abstract class and want to find all implementations.

### prepareCallHierarchy

Initialize call hierarchy analysis for a function. This prepares for incomingCalls or outgoingCalls operations.

### incomingCalls

Find all functions that call a specific function (who calls this?).

```
Operation: incomingCalls
File: src/auth/validate.py
Line: 20
Character: 5

Result:
  - login_user (src/api/auth.py:45) calls validate_token
  - refresh_session (src/api/auth.py:78) calls validate_token
  - middleware_check (src/middleware/auth.py:12) calls validate_token
```

**Use when:** You're changing a function and need to know what might break.

### outgoingCalls

Find all functions that a specific function calls (what does this call?).

```
Operation: outgoingCalls
File: src/api/auth.py
Line: 45
Character: 5

Result:
  - login_user calls:
    - validate_token (src/auth/validate.py:20)
    - create_session (src/auth/session.py:35)
    - log_event (src/utils/logging.py:12)
```

**Use when:** You want to understand a function's dependencies.

## Python-Specific Intelligence

Amplifier uses Pyright for Python language support, providing:

### Type Inference

Even without type annotations, Pyright infers types from your code:

```python
# No annotations here
def get_users(limit):
    return db.query(User).limit(limit).all()

# LSP hover reveals:
# def get_users(limit: int) -> list[User]
```

### Import Resolution

LSP resolves imports correctly, even for:
- Relative imports (`from .utils import helper`)
- Package imports (`from mypackage.submodule import func`)
- Conditional imports
- Dynamic imports (where possible)

### Type Hierarchy

Navigate class inheritance chains:
- Find all subclasses of a base class
- Trace method overrides through inheritance
- Understand mixin compositions

### Stub Files

Pyright uses type stubs (`.pyi` files) for standard library and third-party packages, providing accurate type information even for untyped libraries.

## Best Practices

### 1. Start with LSP for Code Understanding

When investigating unfamiliar code:

1. Use `hover` to understand types and signatures
2. Use `goToDefinition` to see implementations
3. Use `incomingCalls` to understand usage patterns

### 2. Use Call Hierarchy for Impact Analysis

Before refactoring:

1. Run `incomingCalls` on the function you're changing
2. Trace the full call tree to understand blast radius
3. Check all callers to ensure your changes are safe

### 3. Combine with Grep Strategically

Some workflows benefit from both tools:

```
# First, find all files with "authenticate" (fast, broad)
grep pattern="authenticate" type="py"

# Then, use LSP on specific hits for semantic analysis
LSP operation="findReferences" file="src/auth.py" line=45 character=8
```

### 4. Position Matters

LSP operations require precise positions:
- **Line:** 1-indexed (first line is 1)
- **Character:** 1-indexed (first character is 1)
- Position your cursor ON the symbol you're querying

### 5. Delegate Complex Navigation

For multi-step navigation tasks, use specialized agents:
- `lsp:code-navigator` for general code navigation
- `lsp-python:python-code-intel` for Python-specific analysis

## Try It Yourself

### Exercise 1: Trace a Function

Pick any function in your codebase:

1. Use `hover` to see its signature
2. Use `goToDefinition` to find its implementation
3. Use `incomingCalls` to see who calls it
4. Use `outgoingCalls` to see what it depends on

### Exercise 2: Understand a Type

Find a class you're curious about:

1. Use `documentSymbol` to see all its methods
2. Use `goToImplementation` to find subclasses
3. Use `findReferences` on the class name to see usage

### Exercise 3: Impact Analysis

Before making a change:

1. Identify the function you'll modify
2. Use `incomingCalls` recursively to build the call tree
3. Count how many locations might be affected
4. Decide if the change is safe

## Common Errors and Solutions

### "No result found"

**Cause:** LSP couldn't find semantic information at that position.

**Solutions:**
- Ensure cursor is exactly on the symbol (not whitespace)
- Check that the file has valid syntax
- The language server may not have indexed yet—try again

### "Language server not available"

**Cause:** No language server configured for this file type.

**Solutions:**
- Check if the language is supported (Python is built-in)
- Ensure the language server is properly configured
- Some file types don't have LSP support

### "Position out of range"

**Cause:** Line or character number exceeds file bounds.

**Solutions:**
- Remember: lines and characters are 1-indexed
- Verify the file hasn't changed since you got the position
- Double-check your line/character values

### Stale Results

**Cause:** Language server hasn't processed recent changes.

**Solutions:**
- Save the file to trigger re-analysis
- Wait a moment for indexing to complete
- For large changes, the server may need time to catch up

## Summary

The LSP tool provides semantic code intelligence that text search cannot match:

| Capability | What It Does | Key Benefit |
|------------|--------------|-------------|
| `goToDefinition` | Jump to source | No more searching |
| `findReferences` | All usages | Complete picture |
| `hover` | Type info | Understand without reading |
| `incomingCalls` | Who calls this | Impact analysis |
| `outgoingCalls` | What this calls | Dependency mapping |

**Remember:** Use LSP for understanding code relationships. Use grep for finding text patterns. Together, they're a powerful combination for navigating any codebase.
