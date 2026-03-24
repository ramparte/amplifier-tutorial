---
id: lsp-python
type: bundles
title: "LSP Python Bundle"
---

# LSP Python Bundle

## Overview

The LSP Python bundle provides semantic code intelligence for Python codebases using the Language Server Protocol (LSP) powered by Pyright. Unlike text-based search tools like grep, LSP understands your code's actual structure—types, references, call hierarchies, and more.

This bundle transforms how you navigate and understand Python code by providing:

- **Precise symbol lookup**: Find exact definitions, not text matches
- **Semantic references**: Find actual usages, excluding comments and strings
- **Type inference**: Get types even without explicit annotations
- **Call hierarchy**: Trace what calls a function and what it calls

## What's Included

### LSP Tool (Pyright)

The bundle configures and provides access to Pyright, Microsoft's static type checker for Python, as an LSP server. This gives you access to powerful code intelligence operations:

| Operation | Description |
|-----------|-------------|
| `goToDefinition` | Jump to where a symbol is defined |
| `findReferences` | Find all usages of a symbol |
| `hover` | Get type information and documentation |
| `documentSymbol` | List all symbols in a file |
| `workspaceSymbol` | Search for symbols across the workspace |
| `goToImplementation` | Find implementations of abstract methods |
| `prepareCallHierarchy` | Prepare call hierarchy for a symbol |
| `incomingCalls` | Find what calls a function |
| `outgoingCalls` | Find what a function calls |

### python-code-intel Agent

A specialized agent for complex, multi-step Python code navigation tasks. The agent excels at:

- Tracing inheritance chains across multiple files
- Mapping module dependencies
- Understanding type flows through your codebase
- Building comprehensive call graphs
- Debugging type mismatches

## When to Use

### Use LSP Python Bundle When

**Finding where something is defined:**
```
"Where is the Session class defined?"
→ goToDefinition gives exact file:line, not grep's multiple matches
```

**Finding all usages of a symbol:**
```
"What uses the authenticate() method?"
→ findReferences returns actual code usages, ignoring comments/strings
```

**Getting type information:**
```
"What type does get_connection() return?"
→ hover shows inferred types even without annotations
```

**Tracing call relationships:**
```
"What functions call handle_request()?"
→ incomingCalls maps the complete caller graph
```

**Understanding complex codebases:**
```
"Map the authentication flow from login to session creation"
→ python-code-intel agent traces multi-step paths
```

### Use Grep Instead When

- Searching for text patterns (log messages, comments, TODOs)
- Finding configuration values or magic strings
- Bulk text search across many files
- Searching non-Python files

### Quick Decision Guide

| Task | Tool |
|------|------|
| "Find definition of X" | LSP `goToDefinition` |
| "Find all callers of X" | LSP `incomingCalls` |
| "Find text 'ERROR' in logs" | grep |
| "What type is X?" | LSP `hover` |
| "Find TODO comments" | grep |
| "Trace the auth flow" | python-code-intel agent |

## Try It Yourself

### Basic Operations

**1. Go to Definition**

Find where a symbol is defined:

```
LSP operation: goToDefinition
file_path: src/auth/login.py
line: 42
character: 15
```

This returns the exact file and line where the symbol at that position is defined.

**2. Find References**

Find all usages of a symbol:

```
LSP operation: findReferences
file_path: src/models/user.py
line: 10
character: 7
```

Returns every location where `User` (or whatever symbol is at that position) is referenced.

**3. Hover for Type Info**

Get type information and docstrings:

```
LSP operation: hover
file_path: src/utils/cache.py
line: 25
character: 12
```

Returns the inferred type, function signature, and any docstring.

**4. Document Symbols**

List all symbols in a file:

```
LSP operation: documentSymbol
file_path: src/services/payment.py
line: 1
character: 1
```

Returns classes, functions, variables, and their locations in the file.

**5. Workspace Symbol Search**

Search for symbols across the entire workspace:

```
LSP operation: workspaceSymbol
query: "authenticate"
file_path: src/main.py
line: 1
character: 1
```

Finds all symbols matching the query pattern.

### Call Hierarchy Operations

**6. Prepare Call Hierarchy**

Prepare a symbol for call hierarchy analysis:

```
LSP operation: prepareCallHierarchy
file_path: src/api/handlers.py
line: 55
character: 8
```

**7. Incoming Calls**

Find what calls a function (callers):

```
LSP operation: incomingCalls
file_path: src/api/handlers.py
line: 55
character: 8
```

**8. Outgoing Calls**

Find what a function calls (callees):

```
LSP operation: outgoingCalls
file_path: src/api/handlers.py
line: 55
character: 8
```

### Using the python-code-intel Agent

For complex navigation tasks, delegate to the specialized agent:

**Example: Tracing a Feature Flow**

```
Task: "Trace how user authentication works from the login endpoint 
       to session creation, including all intermediate function calls"

Agent: python-code-intel

The agent will:
1. Find the login endpoint definition
2. Trace outgoing calls to identify authentication logic
3. Follow the call chain to session creation
4. Build a complete map of the flow
5. Return a structured summary with file:line references
```

**Example: Finding All Implementations**

```
Task: "Find all classes that implement the PaymentProcessor interface
       and show their process_payment method signatures"

Agent: python-code-intel

The agent will:
1. Locate the PaymentProcessor base class
2. Find all implementations using goToImplementation
3. Extract method signatures using hover
4. Return a comparison of all implementations
```

**Example: Debugging Type Issues**

```
Task: "The function get_user_data() is returning the wrong type somewhere.
       Trace all the places it's called and check the expected vs actual types"

Agent: python-code-intel

The agent will:
1. Get the return type of get_user_data()
2. Find all call sites using incomingCalls
3. Check type expectations at each call site
4. Identify type mismatches
5. Report findings with specific locations
```

## LSP vs Grep: A Practical Comparison

Consider searching for uses of a function called `validate`:

**With grep:**
```
grep -r "validate" src/
```
Returns:
- Actual function calls: `validate(data)`
- Comments: `# TODO: validate input`
- Strings: `"Please validate your email"`
- Other functions: `validate_email()`, `revalidate()`
- Variable names: `is_validated = True`

**With LSP findReferences:**
```
LSP findReferences on validate function
```
Returns:
- Only actual references to that specific function
- No false positives from comments or strings
- No confusion with similarly-named symbols

## Configuration

The LSP Python bundle requires Pyright to be available. It's typically configured in your bundle composition:

```yaml
# Example bundle including LSP Python
extends:
  - lsp-python

# Pyright will use pyrightconfig.json or pyproject.toml
# from your project root for configuration
```

### Pyright Configuration

Create a `pyrightconfig.json` in your project root for best results:

```json
{
  "include": ["src"],
  "exclude": ["**/node_modules", "**/__pycache__", ".venv"],
  "venvPath": ".",
  "venv": ".venv",
  "typeCheckingMode": "basic"
}
```

## Tips for Effective Use

1. **Position matters**: LSP operations work on specific positions. Place the cursor on the symbol you want to analyze.

2. **Let Pyright index first**: Large codebases may need a moment for initial indexing.

3. **Use the agent for multi-step tasks**: Single operations go direct to LSP; complex traces delegate to python-code-intel.

4. **Combine with grep strategically**: Use LSP for semantic queries, grep for text patterns.

5. **Check virtual environments**: Ensure Pyright can find your project's dependencies for accurate type inference.

## Common Patterns

### Pattern: Refactoring Impact Analysis

Before renaming or modifying a function:
1. Use `findReferences` to find all usages
2. Use `incomingCalls` to understand the call graph
3. Use `hover` at each call site to check type expectations

### Pattern: Understanding New Codebases

When exploring unfamiliar code:
1. Use `documentSymbol` to see file structure
2. Use `workspaceSymbol` to find entry points
3. Use `outgoingCalls` to trace execution flow
4. Delegate complex traces to python-code-intel agent

### Pattern: Debugging Type Errors

When facing type mismatches:
1. Use `hover` to get inferred types
2. Use `goToDefinition` to check type definitions
3. Trace the data flow with call hierarchy operations

## Related Bundles

- **lsp**: Base LSP bundle (required dependency)
- **foundation**: Core development tools

## Summary

The LSP Python bundle brings IDE-level code intelligence to your Amplifier workflow. By understanding your code semantically rather than as text, it enables precise navigation, accurate refactoring, and deep code understanding—all accessible through simple tool operations or the specialized python-code-intel agent.
