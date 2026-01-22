---
id: tool-lsp
type: tools
title: "LSP Tool"
---

# LSP Tool (Code Intelligence)

## What is LSP?

The **Language Server Protocol (LSP)** is a standardized protocol that provides semantic code understanding capabilities to editors and development tools. Unlike text-based search tools like `grep`, LSP understands the actual structure and meaning of your code.

LSP enables powerful code intelligence features by:

- **Understanding syntax and semantics**: LSP knows what identifiers are functions, classes, variables, etc.
- **Tracking relationships**: It understands how code elements relate to each other (calls, references, inheritance)
- **Type awareness**: It knows the types of variables and function signatures
- **Cross-file analysis**: It can trace definitions and references across your entire codebase

The LSP tool in Amplifier connects to language servers (like Pyright for Python, TypeScript language server, etc.) to provide semantic code analysis that goes far beyond pattern matching.

## LSP vs Grep

Understanding when to use LSP versus grep is crucial for efficient code navigation:

| Task | Use LSP | Use Grep |
|------|---------|----------|
| Find where function is called | `incomingCalls` - finds actual call sites | Matches any string occurrence, including comments |
| Find function definition | `goToDefinition` - goes to exact definition | Returns multiple matches including false positives |
| Get type information | `hover` - shows type, signature, docs | Not possible with text search |
| Find all references | `findReferences` - semantic references only | Matches variable names in unrelated contexts |
| Understand code structure | `outgoingCalls` - shows call hierarchy | Can't distinguish between different scopes |
| Search across languages | Limited language support | Works on any text file |
| Search in comments/strings | Not designed for this | Perfect for text content |
| Quick pattern matching | Slower, needs language server | Extremely fast for simple patterns |

**Rule of thumb**: Use LSP for semantic code queries, grep for text pattern matching.

## Operations

The LSP tool provides several powerful operations for code intelligence:

### goToDefinition

Jump to where a symbol is defined:

```
lsp(operation="goToDefinition", path="src/main.py", line=10, character=5)
```

**Use cases:**
- Find where a function is implemented
- Locate class definitions
- Jump to variable declarations
- Trace imports to their source

**Returns:** File path, line number, and character position of the definition.

### findReferences

Find all places where a symbol is used:

```
lsp(operation="findReferences", path="src/utils.py", line=15, character=8)
```

**Use cases:**
- Understand how widely a function is used
- Assess impact before refactoring
- Find all usages of a class or variable
- Trace data flow through the codebase

**Returns:** List of all locations where the symbol is referenced, including file, line, and character positions.

### hover

Get detailed information about a symbol at a specific location:

```
lsp(operation="hover", path="src/api.py", line=20, character=12)
```

**Use cases:**
- Check function signatures and parameter types
- View documentation strings
- Understand variable types
- See return type information

**Returns:** Type information, documentation, and signature details for the symbol.

### incomingCalls

Find all functions that call a specific function:

```
lsp(operation="incomingCalls", path="src/service.py", line=50, character=4)
```

**Use cases:**
- Understand the call hierarchy
- Find who depends on your function
- Trace execution flow backwards
- Impact analysis for changes

**Returns:** List of functions that call the target function, with their locations.

### outgoingCalls

Find all functions that a specific function calls:

```
lsp(operation="outgoingCalls", path="src/handler.py", line=30, character=4)
```

**Use cases:**
- Understand what a function depends on
- Trace execution flow forwards
- Map out component dependencies
- Analyze function complexity

**Returns:** List of functions called by the target function, with their locations.

## Python-Specific

The LSP tool uses **Pyright** as the language server for Python code, providing robust type checking and code intelligence.

### Pyright Integration

Pyright offers excellent Python support including:

- **Type inference**: Understands types even without annotations
- **Stub file support**: Uses `.pyi` stubs for better type information
- **Configuration**: Respects `pyproject.toml` and `pyrightconfig.json`
- **Virtual environment detection**: Automatically finds your venv/virtualenv

### Type Checking Levels

Pyright supports different type checking modes:
- **Basic**: Minimal type checking (default for LSP queries)
- **Standard**: Moderate type checking
- **Strict**: Comprehensive type checking

### Common Python LSP Queries

**Finding method implementations in class hierarchies:**
```python
# Use goToDefinition on a method call to find which implementation is actually used
result = obj.process()  # goToDefinition here shows the actual implementation
```

**Tracing decorators:**
```python
# hover over @decorator to see what it does
@cache
@validate_input
def expensive_function():
    pass
```

**Understanding complex types:**
```python
# hover over variables to see inferred types
data = process_items(items)  # hover shows: data: Dict[str, List[Item]]
```

### Python-Specific Tips

1. **Virtual environments**: Ensure your language server can find your venv for accurate import resolution
2. **Type hints**: Add type hints for better LSP results
3. **Docstrings**: LSP shows docstrings in hover information
4. **Imports**: Use absolute imports for better cross-file navigation

## Best Practices

### When to Use LSP

✅ **DO use LSP for:**
- Finding function definitions and implementations
- Understanding call hierarchies
- Checking types and signatures
- Refactoring impact analysis
- Semantic code navigation
- Cross-referencing symbols

❌ **DON'T use LSP for:**
- Searching in comments or documentation
- Finding string literals or patterns
- Searching across non-code files
- Quick text-based searches
- Files without language server support

### Efficient LSP Workflows

**1. Start broad, then narrow:**
```
1. Use findReferences to see all usages
2. Use incomingCalls to understand the call chain
3. Use goToDefinition to examine specific implementations
```

**2. Combine with grep:**
```
1. Use grep to find candidate files/patterns
2. Use LSP to understand semantic relationships
3. Use grep again to verify string patterns if needed
```

**3. Leverage hover for quick insights:**
- Before diving into definitions, use hover to get a quick overview
- Check function signatures before tracing calls
- Understand types before following references

### Performance Considerations

- **Large codebases**: LSP may take time to index initially
- **Language server startup**: First query may be slower
- **Multiple files**: Batch queries when possible
- **Fallback strategy**: If LSP is slow, use grep for initial exploration

### Error Handling

LSP operations may fail if:
- Language server is not available for the file type
- File has syntax errors preventing analysis
- Position is invalid or in a comment
- Symbol cannot be resolved (e.g., dynamic imports)

In these cases, fall back to text-based tools like grep.

## Try It Yourself

### Exercise 1: Navigate a Function Call

1. Find a function definition in your codebase
2. Use `findReferences` to see all its usages
3. Use `incomingCalls` to see what functions call it
4. Use `goToDefinition` on one of the call sites to jump back

### Exercise 2: Understand Type Flow

1. Pick a variable assignment in your code
2. Use `hover` to see its type
3. Use `findReferences` to see where it's used
4. Use `hover` on each usage to see how the type flows

### Exercise 3: Analyze a Refactoring

1. Choose a function you want to rename
2. Use `findReferences` to find all usages
3. Use `incomingCalls` to find all callers
4. Check if any usage is in a critical path
5. Assess the refactoring impact

### Exercise 4: Trace Execution Flow

1. Start with an entry point function
2. Use `outgoingCalls` to see what it calls
3. For each called function, repeat step 2
4. Build a mental map of the execution flow
5. Use `goToDefinition` to examine key functions

## Errors and Troubleshooting

### Common Errors

**Error: "Language server not available"**
- **Cause**: No LSP server configured for this file type
- **Solution**: Ensure the appropriate language server is installed, or use grep instead

**Error: "Symbol not found"**
- **Cause**: Position doesn't point to a valid symbol, or symbol can't be resolved
- **Solution**: Check line/character position, ensure code is valid, try hover first

**Error: "Timeout waiting for language server"**
- **Cause**: Language server is slow or unresponsive
- **Solution**: Wait for indexing to complete, restart session, or use alternative tools

**Error: "Invalid position"**
- **Cause**: Line or character number is out of bounds
- **Solution**: Verify the file content and ensure position is within valid range

### Debugging LSP Issues

**1. Verify file syntax:**
- Ensure the file has no syntax errors
- Language servers can't analyze invalid code

**2. Check position carefully:**
- Line numbers are typically 0-indexed or 1-indexed (check your tool)
- Character position must point to a symbol, not whitespace

**3. Confirm language server status:**
- Check if the language server is running
- Look for initialization errors in logs

**4. Test with hover first:**
- `hover` is the simplest operation
- If hover works, other operations should too

### When LSP Isn't Working

If LSP operations consistently fail:

1. **Fall back to grep**: Use text-based search as a reliable alternative
2. **Check configuration**: Verify language server settings
3. **Use glob**: Find files by pattern, then read them
4. **Manual inspection**: Sometimes reading the code directly is fastest

### Performance Issues

If LSP is too slow:

- **Use grep for initial filtering**: Narrow down candidate files first
- **Avoid repeated queries**: Cache results when possible  
- **Target specific files**: Don't query the entire codebase unnecessarily
- **Consider alternatives**: For large codebases, specialized tools may be faster

## Summary

The LSP tool provides semantic code intelligence that goes far beyond text search. Use it for:

- **Precise navigation**: goToDefinition, findReferences
- **Type understanding**: hover for signatures and types
- **Call analysis**: incomingCalls and outgoingCalls for dependencies
- **Refactoring support**: Impact analysis before changes

Remember: LSP is semantic, grep is textual. Choose the right tool for your task, and combine them for maximum efficiency.

---

**Next Steps:**
- Try the exercises above on your own codebase
- Compare LSP results with grep to understand the difference
- Explore language-specific features for your primary language
- Learn your language server's configuration options for better results
