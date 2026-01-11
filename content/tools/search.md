---
id: tool-search
type: tools
title: "Search Tools"
---

# Search Tools (grep & glob)

Finding files and searching their contents are fundamental operations in any codebase.
Amplifier provides two complementary tools optimized for different search patterns:
**glob** for finding files by name/path, and **grep** for searching file contents.

Understanding when to use each tool will dramatically improve your efficiency.

---

## When to Use Each

| Task | Tool | Why |
|------|------|-----|
| Find files by name | glob | Pattern matching on file paths |
| Search file contents | grep | Regex matching inside files |
| Find all Python files | glob | `**/*.py` matches paths |
| Find function definitions | grep | `def function_name` in content |
| Locate config files | glob | `**/config.*` or `**/*.yaml` |
| Find TODO comments | grep | `TODO\|FIXME` pattern |
| List files in directory | glob | `src/**/*` with path filter |
| Find imports of module | grep | `import module_name` |
| Find test files | glob | `**/test_*.py` or `**/*_test.go` |
| Find error messages | grep | Search for specific strings |

**Rule of Thumb:**
- Know the filename pattern? Use **glob**
- Know what's inside the file? Use **grep**
- Need both? Combine them (see below)

---

## Glob Patterns

Glob uses shell-style pattern matching to find files by their paths.

### Basic Patterns

| Pattern | Matches | Example |
|---------|---------|---------|
| `*` | Any characters in filename | `*.py` matches `main.py`, `test.py` |
| `**` | Any directory depth | `**/*.md` matches `docs/api/ref.md` |
| `?` | Single character | `file?.txt` matches `file1.txt` |
| `[abc]` | Character set | `[Rr]eadme.md` matches both cases |
| `[!abc]` | Negated set | `[!.]*.py` excludes dotfiles |

### Common Use Cases

```
# All Python files in project
**/*.py

# All files in src directory (one level)
src/*

# All files in src directory (recursive)
src/**/*

# Configuration files
**/config.* 
**/*.yaml
**/*.json

# Test files (Python convention)
**/test_*.py
**/*_test.py

# TypeScript and JavaScript
**/*.{ts,tsx,js,jsx}

# Markdown documentation
docs/**/*.md

# Hidden files
**/.*

# Package manifests
**/package.json
**/pyproject.toml
**/Cargo.toml
```

### Glob Parameters

```python
glob(
    pattern="**/*.py",      # Required: the pattern to match
    path="src",             # Base directory (default: current)
    type="file",            # "file", "dir", or "any"
    exclude=["test_*"],     # Patterns to exclude
    include_ignored=False   # Search in .gitignore'd dirs
)
```

### Examples

```python
# Find all Python files
glob(pattern="**/*.py")

# Find directories named "tests"
glob(pattern="**/tests", type="dir")

# Find configs, excluding node_modules
glob(pattern="**/*.config.js", exclude=["node_modules/**"])

# Search in normally-ignored directories
glob(pattern="**/*.py", include_ignored=True)
```

---

## Grep Patterns

Grep searches file contents using regular expressions (regex).

### Basic Regex

| Pattern | Matches | Example |
|---------|---------|---------|
| `.` | Any single character | `a.c` matches `abc`, `a1c` |
| `*` | Zero or more of previous | `ab*c` matches `ac`, `abc`, `abbc` |
| `+` | One or more of previous | `ab+c` matches `abc`, `abbc` |
| `?` | Zero or one of previous | `ab?c` matches `ac`, `abc` |
| `^` | Start of line | `^import` matches at line start |
| `$` | End of line | `\);$` matches line-ending `;` |
| `\s` | Whitespace | `def\s+\w+` matches function defs |
| `\w` | Word character | `\w+` matches identifiers |
| `\b` | Word boundary | `\bclass\b` avoids `subclass` |

### Common Search Patterns

```regex
# Function definitions (Python)
def\s+\w+\s*\(

# Class definitions (Python)
class\s+\w+

# Import statements
^import\s+|^from\s+\w+\s+import

# TODO/FIXME comments
TODO|FIXME|XXX|HACK

# Console/print statements
console\.(log|error|warn)|print\(

# Error handling
except\s+\w+|catch\s*\(|\.catch\(

# API endpoints
@(app|router)\.(get|post|put|delete)

# Environment variables
os\.environ|process\.env

# Hardcoded URLs
https?://[^\s"']+

# SQL-like patterns (potential injection)
(SELECT|INSERT|UPDATE|DELETE)\s+
```

### Grep Parameters

```python
grep(
    pattern="def\\s+\\w+",   # Required: regex pattern
    path="src",              # Directory to search
    glob="*.py",             # Filter by filename pattern
    type="py",               # Or use file type shorthand
    output_mode="content",   # How to display results
    head_limit=100,          # Limit results
    "-i": True,              # Case insensitive
    "-n": True,              # Show line numbers
    "-A": 2,                 # Lines after match
    "-B": 2,                 # Lines before match
    "-C": 3,                 # Lines of context (before+after)
    multiline=True,          # Match across lines
    include_ignored=False    # Search ignored directories
)
```

---

## Output Modes

Grep supports three output modes for different use cases:

### files_with_matches (default)

Returns only file paths that contain matches. Fast and concise.

```python
grep(pattern="async def", output_mode="files_with_matches")
```

Output:
```
src/api/handlers.py
src/services/processor.py
src/utils/async_helpers.py
```

**Best for:** Quick discovery, finding which files to examine.

### content

Returns matching lines with context. Most detailed output.

```python
grep(pattern="class.*Error", output_mode="content", "-n": True, "-C": 2)
```

Output:
```
src/exceptions.py
   12:
   13: # Custom application errors
   14: class ValidationError(Exception):
   15:     """Raised when input validation fails."""
   16:     pass
```

**Best for:** Understanding context, code review, detailed analysis.

### count

Returns match counts per file. Great for metrics.

```python
grep(pattern="TODO", output_mode="count")
```

Output:
```
src/api/routes.py: 3
src/models/user.py: 1
src/services/auth.py: 7
```

**Best for:** Codebase metrics, prioritizing refactoring, finding hotspots.

---

## Combining Tools

The most powerful searches combine glob and grep.

### Pattern 1: Filter by File Type Then Search

```python
# Find all config files, then search for database settings
files = glob(pattern="**/*.yaml")
grep(pattern="database:", path=".", glob="*.yaml")
```

### Pattern 2: Use Grep's Built-in Filtering

```python
# Search only in Python files
grep(pattern="import requests", type="py")

# Search only in specific directory with glob filter
grep(pattern="async def", path="src/api", glob="*.py")
```

### Pattern 3: Progressive Refinement

```python
# Step 1: Find all test files
glob(pattern="**/test_*.py")

# Step 2: Find tests that use mocking
grep(pattern="@patch|Mock\(", glob="**/test_*.py")

# Step 3: Find specific mock patterns with context
grep(pattern="@patch.*database", glob="**/test_*.py", 
     output_mode="content", "-C": 3)
```

### Pattern 4: Cross-Reference

```python
# Find where a class is defined
grep(pattern="^class UserService", type="py")
# Result: src/services/user.py

# Find all usages of that class
grep(pattern="UserService", type="py", output_mode="content")
```

---

## Best Practices

### 1. Start Broad, Then Narrow

```python
# Too specific - might miss variations
grep(pattern="def getUserById")

# Better - find all user-related functions
grep(pattern="def.*[Uu]ser", type="py")
```

### 2. Use Type Filters for Speed

```python
# Slower - searches all files
grep(pattern="import", path=".")

# Faster - only Python files
grep(pattern="import", type="py")
```

### 3. Limit Results to Avoid Overload

```python
# Could return thousands of matches
grep(pattern="def ")

# Better - limit and paginate
grep(pattern="def ", head_limit=50, offset=0)
```

### 4. Use Word Boundaries for Precision

```python
# Matches "class", "subclass", "classification"
grep(pattern="class")

# Matches only "class" keyword
grep(pattern="\\bclass\\b")
```

### 5. Escape Special Characters

```python
# Wrong - . matches any character
grep(pattern="config.yaml")

# Correct - \. matches literal dot
grep(pattern="config\\.yaml")
```

### 6. Use Case Insensitivity When Appropriate

```python
# Find README regardless of case
grep(pattern="readme", "-i": True)
```

---

## Try It Yourself

### Exercise 1: Find All API Routes

Find files that define HTTP endpoints in a Python Flask/FastAPI project.

```python
# Your solution:
grep(pattern="@(app|router)\\.(get|post|put|delete|patch)", type="py")
```

### Exercise 2: Locate Configuration Files

Find all YAML and JSON configuration files, excluding test fixtures.

```python
# Your solution:
glob(pattern="**/*.{yaml,yml,json}", exclude=["**/fixtures/**", "**/test/**"])
```

### Exercise 3: Find Unused Imports

Search for import statements that might be unused (import but no usage).

```python
# Step 1: Find all imports
grep(pattern="^from\\s+\\w+\\s+import\\s+(\\w+)", type="py", output_mode="content")

# Step 2: For each import, search for usages (manual verification needed)
```

### Exercise 4: Count TODOs by Directory

Get a count of TODO comments to prioritize cleanup.

```python
grep(pattern="TODO|FIXME", output_mode="count")
```

### Exercise 5: Find Large Functions

Find function definitions and use context to assess complexity.

```python
grep(pattern="^\\s*def\\s+\\w+", type="py", output_mode="content", "-A": 30)
```

---

## Common Errors and Solutions

### Error: "Pattern not found"

**Cause:** Pattern doesn't match any content, or wrong file type filter.

**Solution:**
```python
# Check if files exist first
glob(pattern="**/*.py")

# Try broader pattern
grep(pattern="user", type="py")  # Instead of "UserService"
```

### Error: "Too many results"

**Cause:** Pattern too broad, missing filters.

**Solution:**
```python
# Add file type filter
grep(pattern="import", type="py")

# Limit results
grep(pattern="import", head_limit=100)

# Narrow the path
grep(pattern="import", path="src/api")
```

### Error: "Regex syntax error"

**Cause:** Unescaped special characters or invalid regex.

**Solution:**
```python
# Escape special characters
grep(pattern="\\[.*\\]")     # Match brackets
grep(pattern="\\$\\w+")      # Match $variables
grep(pattern="file\\.txt")   # Match literal dot
```

### Error: "Results include unwanted directories"

**Cause:** Searching in node_modules, .venv, etc.

**Solution:**
```python
# These are excluded by default, but if you used include_ignored:
grep(pattern="import", include_ignored=False)  # Default behavior

# Or use explicit exclusions in glob
glob(pattern="**/*.py", exclude=["**/venv/**", "**/.venv/**"])
```

---

## Quick Reference

| Operation | Command |
|-----------|---------|
| Find Python files | `glob(pattern="**/*.py")` |
| Find in directory | `glob(pattern="src/**/*")` |
| Search content | `grep(pattern="def main")` |
| Case insensitive | `grep(pattern="error", "-i": True)` |
| With context | `grep(pattern="error", "-C": 3)` |
| Count matches | `grep(pattern="TODO", output_mode="count")` |
| Files only | `grep(pattern="class", output_mode="files_with_matches")` |
| Filter by type | `grep(pattern="import", type="py")` |

---

## Summary

- **glob**: Find files by path patterns (`**/*.py`, `src/**/*.ts`)
- **grep**: Search file contents with regex (`def\s+\w+`, `TODO|FIXME`)
- **Combine them**: Use glob to find files, grep to search contents
- **Output modes**: files_with_matches (fast), content (detailed), count (metrics)
- **Best practice**: Start broad, filter by type, limit results, use word boundaries
