---
id: tool-search
type: tools
title: "Search Tools"
---

# Search Tools (grep & glob)

Master the art of finding files and searching content across your codebase. The `grep` and `glob` tools are your primary search capabilities for navigating projects of any size.

## When to Use Each

| Task | Tool | Why |
|------|------|-----|
| Find files by name/extension | `glob` | Fast pattern matching on file paths |
| Search inside file contents | `grep` | Regex-based content search |
| Find all Python files | `glob` | Use `**/*.py` pattern |
| Find TODO comments | `grep` | Search for `TODO:` in code |
| List all test files | `glob` | Match `**/test_*.py` or `*.test.js` |
| Find function definitions | `grep` | Regex like `function\s+\w+` |
| Locate config files | `glob` | Match `**/*.{json,yaml,toml}` |
| Find import statements | `grep` | Search for `import.*Module` |

**Quick Decision Guide:**
- Know the filename? → Use `glob`
- Know what's *inside* the file? → Use `grep`
- Need both? → Combine them!

## Glob Patterns

Glob finds files by matching path patterns. Think of it as a smart file finder.

### Basic Patterns

```bash
# Find all Python files in current directory
*.py

# Find all Python files recursively
**/*.py

# Find all JavaScript and TypeScript files
**/*.{js,ts}

# Find all test files
**/test_*.py
**/*_test.go
**/*.test.js
```

### Pattern Syntax

| Pattern | Matches | Example |
|---------|---------|---------|
| `*` | Any characters (except `/`) | `*.md` → `README.md` |
| `**` | Any directories recursively | `**/src/*.js` |
| `?` | Single character | `test?.py` → `test1.py` |
| `[abc]` | Any character in brackets | `file[123].txt` |
| `{a,b}` | Either a or b | `*.{js,ts}` |

### Common Use Cases

**Find all source files:**
```bash
# Python project
**/*.py

# JavaScript/TypeScript project
**/*.{js,ts,jsx,tsx}

# Rust project
**/*.rs
```

**Find configuration files:**
```bash
**/*.{json,yaml,yml,toml,ini}
```

**Find documentation:**
```bash
**/*.md
**/docs/**/*.md
```

**Find specific directories:**
```bash
# Find all __init__.py files (Python packages)
**/__init__.py

# Find all package.json files (Node modules)
**/package.json
```

### Glob Parameters

```yaml
pattern: "**/*.py"           # Required: glob pattern
path: "./src"                # Optional: base directory (default: current)
type: "file"                 # Optional: file, dir, or any (default: file)
include_ignored: false       # Optional: search in node_modules, .venv, etc.
exclude: ["**/test_*.py"]    # Optional: patterns to exclude
```

### Glob Output

Results are sorted by modification time (newest first) and limited to 500 files:

```
Files matched (250 of 500+ total):
  src/main.py
  src/utils.py
  tests/test_main.py
  ...

Note: Results limited to 500 files. 234 additional files not shown.
```

## Grep Patterns

Grep searches *inside* files using regular expressions. It's your content detective.

### Basic Patterns

```bash
# Find TODO comments
TODO

# Find function definitions (JavaScript)
function\s+\w+

# Find class definitions (Python)
class\s+\w+

# Find import statements
import.*requests

# Find error handling
except\s+\w+Error

# Case insensitive search
-i: true
pattern: "error"
```

### Regex Syntax

Grep uses ripgrep regex (Rust flavor):

| Pattern | Matches | Example |
|---------|---------|---------|
| `.` | Any character | `log.` → `log1`, `logA` |
| `\d` | Any digit | `user\d+` → `user123` |
| `\w` | Word character | `\w+` → `hello` |
| `\s` | Whitespace | `if\s+` → `if ` |
| `*` | 0 or more | `error.*` → `error: failed` |
| `+` | 1 or more | `\d+` → `123` |
| `?` | 0 or 1 | `colou?r` → `color`, `colour` |
| `^` | Start of line | `^import` → import at start |
| `$` | End of line | `;$` → semicolon at end |
| `[abc]` | Character class | `[Ee]rror` → Error, error |
| `(a\|b)` | Alternation | `(get\|set)` → get or set |

**Important:** Ripgrep requires escaping braces: `interface\{` to find `interface{`

### Output Modes

Grep has three output modes for different needs:

#### 1. files_with_matches (Default)

Lists files containing matches (fastest):

```yaml
pattern: "TODO"
output_mode: "files_with_matches"
```

Output:
```
src/main.py
src/utils.py
tests/test_main.py
```

**Use when:** You just need to know *which* files contain something.

#### 2. content

Shows matching lines with content:

```yaml
pattern: "def\s+\w+"
output_mode: "content"
```

Output:
```
src/main.py:
  15: def process_data(input):
  42: def validate_input(data):

src/utils.py:
  8: def helper_function():
```

**Use when:** You need to see the actual matches.

#### 3. count

Shows count of matches per file:

```yaml
pattern: "import"
output_mode: "count"
```

Output:
```
src/main.py: 15 matches
src/utils.py: 8 matches
tests/test_main.py: 12 matches
```

**Use when:** You need statistics or metrics.

### Content Mode Features

When using `output_mode: "content"`, you get extra options:

**Line numbers** (default: true):
```yaml
-n: true  # Show line numbers
```

**Context lines:**
```yaml
-A: 2  # Show 2 lines AFTER match
-B: 2  # Show 2 lines BEFORE match
-C: 2  # Show 2 lines before AND after (context)
```

Example with context:
```yaml
pattern: "raise ValueError"
output_mode: "content"
-C: 2
```

Output:
```
src/main.py:
  28:     if not valid:
  29:         logger.error("Invalid input")
  30:         raise ValueError("Input validation failed")
  31:         return None
  32:     process(data)
```

### File Filtering

**By file type:**
```yaml
pattern: "TODO"
type: "py"  # Search only Python files
```

Supported types: `py`, `js`, `ts`, `go`, `rust`, `java`, `c`, `cpp`, `rb`, `sh`, `md`, `json`, `yaml`, `html`, `css`, `xml`, `php`, `sql`, `swift`, `lua`, `toml`, `txt`

**By glob pattern:**
```yaml
pattern: "TODO"
glob: "**/*.{js,ts}"  # Search only JS/TS files
```

**By path:**
```yaml
pattern: "test"
path: "./tests"  # Search only in tests directory
```

**Include ignored directories:**
```yaml
pattern: "config"
include_ignored: true  # Search in node_modules, .venv, etc.
```

By default, grep excludes: `node_modules`, `.venv`, `.git`, `__pycache__`, `build`, `dist`, `.next`, `target`

### Multiline Patterns

Search across multiple lines:

```yaml
pattern: "function.*\\{.*\\}"
multiline: true
```

This allows `.` to match newlines and patterns to span multiple lines.

### Pagination

Handle large result sets:

```yaml
pattern: "import"
head_limit: 100      # Show first 100 results
offset: 0            # Start from beginning

# For next page:
head_limit: 100
offset: 100          # Skip first 100
```

The response includes `total_matches` so you know how many exist.

## Combining Tools

The real power comes from using both tools together!

### Pattern 1: Narrow Down Files First

1. Use `glob` to find candidate files
2. Use `grep` to search within those files

```yaml
# Step 1: Find all TypeScript files
glob: "**/*.ts"

# Step 2: Search for React hooks in those files
pattern: "use[A-Z]\\w+"
glob: "**/*.ts"
```

### Pattern 2: Search Then Explore

1. Use `grep` to find files with matches
2. Read those files to see context

```yaml
# Step 1: Find files with TODO comments
pattern: "TODO"
output_mode: "files_with_matches"

# Step 2: Read specific files
# (Use read_file tool on results)
```

### Pattern 3: Iterative Refinement

Start broad, narrow down:

```yaml
# Round 1: Find all error handling
pattern: "except"
type: "py"

# Round 2: Find specific exception types
pattern: "except\\s+(ValueError|TypeError)"
output_mode: "content"

# Round 3: Add context to see full try/except blocks
pattern: "except\\s+(ValueError|TypeError)"
output_mode: "content"
-C: 5
```

## Best Practices

### Performance Tips

1. **Use file type filters**: `type: "py"` is faster than searching all files
2. **Specify paths**: Searching `./src` is faster than entire project
3. **Start with files_with_matches**: Cheapest output mode
4. **Use head_limit**: Limit results to prevent overwhelming context

### Search Strategy

**For unknown codebases:**
```
1. glob → Find what files exist
2. grep (files_with_matches) → Find relevant files
3. grep (content) → See actual code
4. read_file → Read full context
```

**For specific searches:**
```
1. Combine grep with type/glob filters immediately
2. Use content mode with context (-C)
3. Read promising files
```

### Common Patterns

**Find API endpoints:**
```yaml
pattern: "@(app|router)\\.(get|post|put|delete)"
type: "py"
output_mode: "content"
```

**Find environment variables:**
```yaml
pattern: "os\\.environ|process\\.env"
output_mode: "content"
```

**Find security issues:**
```yaml
pattern: "(eval|exec|system|shell_exec)\\("
output_mode: "content"
-C: 3
```

**Find deprecated code:**
```yaml
pattern: "@deprecated|DEPRECATED|FIXME"
output_mode: "content"
```

## Try It Yourself

### Exercise 1: Find All Tests

Find all test files in your project:

```yaml
# Glob approach:
pattern: "**/test_*.py"
```

### Exercise 2: Find Unused Imports

Search for imports that might be unused:

```yaml
pattern: "^import\\s+\\w+"
type: "py"
output_mode: "content"
```

### Exercise 3: Configuration Audit

Find all configuration values:

```yaml
# Find config files
pattern: "**/*.{json,yaml,yml,toml}"

# Then search for secrets/credentials
pattern: "(password|secret|api_key|token)"
-i: true
output_mode: "content"
```

### Exercise 4: Code Complexity

Find long functions (might need refactoring):

```yaml
# Find function definitions
pattern: "^(def|function)\\s+\\w+"
output_mode: "content"
-A: 50  # Show next 50 lines to see function size
```

## Common Errors

### Glob Errors

**"No files matched pattern"**
- Check your pattern syntax
- Try simpler patterns first: `*.py` before `**/*.py`
- Verify you're in the right directory
- Use `include_ignored: true` if searching excluded dirs

**"Too many results (>500)"**
- Be more specific with your pattern
- Search in a subdirectory: `path: "./src"`
- Use exclude patterns: `exclude: ["**/node_modules/**"]`

### Grep Errors

**"Pattern not found"**
- Verify regex syntax (ripgrep flavor)
- Try case insensitive: `-i: true`
- Check file type filter: `type: "py"`
- Use simpler pattern first, then refine

**"Results truncated"**
- This is normal! Results are limited for performance
- Use `head_limit` and `offset` for pagination
- Consider narrowing search with `path` or `type`

**"Regex syntax error"**
- Escape special characters: `\{`, `\(`, `\.`
- Use raw strings in your mind: `\\` for backslash
- Test simpler patterns first

**"Multiline not working"**
- Set `multiline: true` explicitly
- Remember `.` now matches newlines
- Patterns can span lines: `function.*\\{.*\\}`

### Pro Tips

1. **Test patterns incrementally**: Start simple, add complexity
2. **Use files_with_matches first**: See what files match before getting content
3. **Combine with read_file**: Grep finds it, read_file shows full context
4. **Watch output limits**: Default limits prevent context overflow
5. **Case sensitivity matters**: Use `-i: true` when unsure
6. **Escape ripgrep regex**: `\{` `\}` `\(` `\)` need escaping

---

**Next Steps:**
- Practice with your own codebase
- Combine with other tools like `read_file` and `bash`
- Learn advanced regex patterns
- Build search workflows for common tasks

**Related Tools:**
- `read_file` - Read full file content after finding matches
- `bash` - Use shell commands like `find` or `ag` if needed
- `task:explorer` - For complex multi-step searches
