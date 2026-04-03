---
id: tool-search
type: tool
title: "Search Tools"
---

# Search Tools

You have two tools for finding things in a codebase: `grep` searches *inside* files for content that matches a pattern, and `glob` finds files *by name*. Together they cover every "where is that thing?" question — from "which file defines this function?" to "show me every Python file under src/."

## What Are grep and glob?

Think of it this way: `glob` is your file explorer's search bar, and `grep` is Ctrl+F across every file at once.

| Tool | What It Searches | Think of It As |
|------|-----------------|----------------|
| `grep` | Text inside files | "Find in Files" |
| `glob` | File and directory names | "Find File by Name" |

**Quick decision:** If you know the filename, use `glob`. If you know what's *written* in the file, use `grep`. If you need both — find the files first with `glob`, then search inside them with `grep`.

## Core Capabilities: glob

`glob` matches file paths against patterns. You give it a pattern like `**/*.py` and it returns every Python file in the project:

```
> Find all the Python files in this project

[Tool: glob] **/*.py
  src/main.py
  src/config.py
  src/utils/helpers.py
  tests/test_main.py
  tests/test_utils.py
  (5 files, sorted by modification time)
```

Results come back sorted by modification time — newest first — so the file you just touched is at the top. Common non-source directories (`node_modules`, `.venv`, `.git`, `__pycache__`) are automatically excluded, and results cap at 500 files.

You can narrow the search with a base path, filter by type, or reach into excluded directories:

```
> Find all directories under src/

[Tool: glob] src/*
  type: dir
  src/utils/
  src/models/
  src/api/
```

Common patterns:

```
**/*.py              All Python files, recursively
**/*.{js,ts,tsx}     JavaScript and TypeScript files
**/test_*.py         Python test files
**/Dockerfile        All Dockerfiles
**/*.md              All Markdown files
```

## Core Capabilities: grep

`grep` searches file contents using regular expressions. It's powered by ripgrep under the hood — fast, respects exclusions, and handles large codebases without breaking a sweat.

### Finding matches

The simplest use: search for a string and see which files contain it.

```
> Where are the TODO comments in this project?

[Tool: grep] pattern: "TODO"
  src/main.py
  src/utils/helpers.py
  tests/test_main.py
```

That's the default `files_with_matches` mode — the fastest way to answer "which files mention this?"

### Seeing content

When you need to see the actual matching lines, switch to `content` mode:

```
> Show me every function definition in the Python source

[Tool: grep] pattern: "def \w+", type: py, output_mode: content
  src/main.py:
    15: def process_data(input):
    42: def validate_input(data):
  src/utils/helpers.py:
    8: def format_date(dt):
    23: def retry(func, attempts=3):
```

### Context lines

Seeing a matching line is useful; seeing what's *around* it is often better. Use `-A` (after), `-B` (before), or `-C` (both) to include surrounding lines:

```
> Show me every raise ValueError with context

[Tool: grep] pattern: "raise ValueError", output_mode: content, -C: 2
  src/main.py:
    28:     if not valid:
    29:         logger.error("Invalid input")
    30:         raise ValueError("Input validation failed")
    31:     return None
    32:     process(data)
```

### Counting matches

When you want statistics rather than content, use `count` mode:

```
> How many import statements are in each file?

[Tool: grep] pattern: "^import", type: py, output_mode: count
  src/main.py: 12 matches
  src/utils/helpers.py: 5 matches
  tests/test_main.py: 8 matches
```

### Filtering what you search

You can narrow `grep` searches three ways:

- **By file type** — `type: "py"` searches only Python files. Supports `py`, `js`, `ts`, `go`, `rust`, `java`, `c`, `cpp`, `rb`, `sh`, `md`, `json`, `yaml`, and many more.
- **By glob pattern** — `glob: "src/**/*.{js,ts}"` limits to matching paths.
- **By directory** — `path: "./tests"` searches only within that subtree.

```
> Find TODO comments, but only in the test files

[Tool: grep] pattern: "TODO", path: "./tests", type: py
  tests/test_main.py
  tests/test_integration.py
```

Like `glob`, `grep` automatically excludes `node_modules`, `.venv`, `.git`, `__pycache__`, and build directories. Set `include_ignored: true` to override that.

### Case-insensitive and multiline

Add `-i: true` for case-insensitive matching — useful when you're not sure whether someone wrote `Error`, `ERROR`, or `error`.

For patterns that span multiple lines, set `multiline: true`. This lets `.` match newlines so you can find multi-line structures like decorators followed by function definitions.

### Pagination

Large result sets are capped by default (200 files, 500 content matches). Use `head_limit` and `offset` to page through them:

```
> Show me the next batch of results

[Tool: grep] pattern: "import", head_limit: 100, offset: 100
```

The response includes `total_matches` so you always know how many exist beyond what's shown.

## Practical Examples

### Exploring an unfamiliar project

Start broad with `glob`, then drill in with `grep`:

```
> I just cloned this repo. What are we working with?

[Tool: glob] **/*.py
  (42 files found)

[Tool: glob] **/*.{json,yaml,toml}
  pyproject.toml
  config/settings.yaml

[Tool: grep] pattern: "def main", type: py, output_mode: content
  src/main.py:
    8: def main():
```

### Finding all callers of a function

```
> Where is process_data called?

[Tool: grep] pattern: "process_data\(", type: py, output_mode: content
  src/main.py:
    45:     result = process_data(raw_input)
  src/api/handlers.py:
    22:     return process_data(request.body)
  tests/test_main.py:
    15:     assert process_data("hello") == expected
```

### Combining glob and grep

The classic two-step: find the right files, then search within them.

```
> Are there any hardcoded passwords in the config files?

[Tool: glob] **/*.{yaml,json,toml,env}
  config/settings.yaml
  config/prod.yaml
  .env.example

[Tool: grep] pattern: "(password|secret|api_key)", -i: true, glob: "**/*.{yaml,json,toml,env}", output_mode: content
  config/settings.yaml:
    12: database_password: "${DB_PASSWORD}"
  .env.example:
    3: API_KEY=your-key-here
```

## grep vs LSP: When to Use Which

Amplifier also has [LSP (Code Intelligence)](./lsp.md) tools that understand code *semantically* — types, definitions, call hierarchies. Here's when each is the right choice:

| Task | Best Tool | Why |
|------|-----------|-----|
| Find callers of a function | **LSP** `incomingCalls` | Semantic — ignores comments and strings |
| Find where a symbol is defined | **LSP** `goToDefinition` | Jumps straight to the definition |
| Get type info or signature | **LSP** `hover` | grep can't infer types |
| Rename a symbol safely | **LSP** `rename` | Cross-file, semantically aware |
| Find a text pattern anywhere | **grep** | Faster, works on any file type |
| Search across many files at once | **grep** | Bulk text search is grep's strength |
| Find TODO/FIXME comments | **grep** | Text matching, not code analysis |
| Search non-code files (docs, config) | **grep** | LSP only covers source code |

The rule of thumb: use LSP when you need to understand code *meaning* (definitions, types, usages). Use `grep` when you need to find *text* (patterns, comments, configuration values).

## Tips

**Start with `files_with_matches`.** It's the cheapest output mode. See which files match first, then switch to `content` for the ones that matter.

**Filter early.** Searching `path: "./src"` with `type: "py"` is much faster than searching the entire project tree. Narrow the scope before you search.

**Use context lines liberally.** A bare match line rarely tells the whole story. `-C: 3` costs almost nothing and often saves a follow-up `read_file` call.

**Combine, don't replace.** `glob` and `grep` are teammates. Use `glob` to orient yourself ("what files exist?"), then `grep` to find what's inside them.

**Escape ripgrep regex.** Literal braces need escaping: `interface\{` to match `interface{`. Parentheses, dots, and brackets are regex metacharacters too.

**Don't forget `-i`.** Case-insensitive search catches `Error`, `ERROR`, and `error` in one pass. Use it when you're not sure about casing.

## Next Steps

- Learn about [Filesystem Tools](./filesystem.md) for reading and editing the files you find
- See [LSP (Code Intelligence)](./lsp.md) for semantic navigation — definitions, references, and rename
- Explore [Bash Tool](./bash.md) for running tests and builds after making changes
