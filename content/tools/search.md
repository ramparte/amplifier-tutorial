---
id: tool-search
type: tools
title: "Search Tools (grep & glob)"
---

# Search Tools

Find files and search content with `grep` and `glob`.

## Overview

| Tool | Purpose | Use When |
|------|---------|----------|
| `grep` | Search file contents | Find code patterns, text |
| `glob` | Find files by name | Locate files, explore structure |

## grep - Search Contents

Search for patterns inside files.

### Basic Search

```
> Find all TODO comments
```

```
[Tool: grep]
pattern: "TODO"
```

### Search Specific Path

```
> Find all print statements in src/
```

```
[Tool: grep]
pattern: "print\\("
path: src/
```

### Regex Patterns

```
# Function definitions
pattern: "def \\w+\\("

# Import statements
pattern: "^import|^from .* import"

# Email addresses
pattern: "[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}"
```

### File Type Filter

```
> Find 'config' only in Python files
```

```
[Tool: grep]
pattern: "config"
type: py
```

Available types: `py`, `js`, `ts`, `rust`, `go`, `java`, `md`, etc.

### Case Insensitive

```
[Tool: grep]
pattern: "error"
-i: true
```

### Show Context

```
# Show 2 lines before and after each match
[Tool: grep]
pattern: "raise Exception"
-C: 2
output_mode: content
```

### Count Matches

```
[Tool: grep]
pattern: "import"
output_mode: count
```

### Output Modes

| Mode | Returns |
|------|---------|
| `files_with_matches` | List of matching files (default) |
| `content` | Matching lines with context |
| `count` | Number of matches per file |

## glob - Find Files

Find files by name pattern.

### Basic Patterns

```
# All Python files
> Find all Python files
[Tool: glob]
pattern: "**/*.py"

# All test files
> Find test files
[Tool: glob]
pattern: "**/test_*.py"

# Config files
> Find config files
[Tool: glob]
pattern: "**/*.{json,yaml,yml}"
```

### Pattern Syntax

| Pattern | Matches |
|---------|---------|
| `*` | Any characters (not /) |
| `**` | Any path (including /) |
| `?` | Single character |
| `[abc]` | Character set |
| `{a,b}` | Alternatives |

### Examples

```
# All markdown in docs/
pattern: "docs/**/*.md"

# All index files
pattern: "**/index.{js,ts}"

# Single character variant
pattern: "file?.txt"  # file1.txt, fileA.txt

# Character range
pattern: "log[0-9].txt"  # log1.txt, log2.txt
```

### Exclude Patterns

```
[Tool: glob]
pattern: "**/*.py"
exclude: ["**/__pycache__/**", "**/venv/**"]
```

### Filter by Type

```
# Only directories
[Tool: glob]
pattern: "src/*"
type: dir

# Only files
[Tool: glob]
pattern: "src/*"
type: file
```

## Common Workflows

### Find and Read

```
> Find the authentication module and show me its contents
```

1. `[glob] pattern: "**/auth*.py"` → finds `src/auth.py`
2. `[read_file] file_path: src/auth.py`

### Find and Edit

```
> Find all files with deprecated API and update them
```

1. `[grep] pattern: "deprecated_function"` → lists files
2. `[edit_file]` each file with replacement

### Explore Structure

```
> What's the project structure?
```

```
[glob] pattern: "**/*" type: dir
```

### Find Definition

```
> Where is the User class defined?
```

```
[grep] pattern: "class User"
```

## Excluded Directories

By default, these are excluded:
- `node_modules/`
- `.venv/`, `venv/`
- `.git/`
- `__pycache__/`
- `build/`, `dist/`

To include them:

```
[Tool: grep]
pattern: "lodash"
include_ignored: true
```

## Try It Yourself

### Exercise 1: Find TODOs

```
> Find all TODO and FIXME comments in the project
```

### Exercise 2: Explore Structure

```
> List all directories under src/
```

### Exercise 3: Find Usage

```
> Find all files that import the 'requests' library
```

### Exercise 4: Complex Pattern

```
> Find all functions that start with 'get_' or 'fetch_'
```

## grep vs bash grep

Prefer Amplifier's grep tool over bash:

| Amplifier grep | bash grep |
|----------------|-----------|
| Structured output | Raw text |
| Smart exclusions | Manual excludes |
| Output limits | May overflow |
| Integrated results | Separate parsing |

```
# Prefer this:
[Tool: grep] pattern: "TODO"

# Over this:
[bash] grep -r "TODO" .
```

## Errors and Troubleshooting

### "No matches found"

- Check pattern syntax (escape special chars)
- Check path is correct
- Try case-insensitive with `-i: true`

### "Too many results"

- Add path filter
- Use file type filter
- Add more specific pattern

### Regex Escape

Special characters need escaping:

```
# Search for literal "function()"
pattern: "function\\(\\)"

# Search for file path
pattern: "src/utils\\.py"
```
