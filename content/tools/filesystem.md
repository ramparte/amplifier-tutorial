---
id: tool-filesystem
type: tool
title: "Filesystem Tools"
---

# Filesystem Tools

Every meaningful task eventually comes down to files — reading code to understand it, editing a function to fix a bug, creating a new module from scratch, or finding the right file in a sprawling project. Amplifier gives you five tools for this: `read_file`, `write_file`, `edit_file`, `apply_patch`, and `glob`. Each one fits a different shape of work, and knowing which to reach for is half the battle.

## What Are the Filesystem Tools?

Think of them as a toolkit with different levels of precision:

| Tool | What It Does | Think of It As |
|------|-------------|----------------|
| `read_file` | Read a file's contents or list a directory | Opening a file in your editor |
| `write_file` | Create a new file or overwrite an existing one | "Save As" |
| `edit_file` | Replace a specific string inside a file | Find-and-replace |
| `apply_patch` | Create, update, or delete multiple files at once | A multi-file commit |
| `glob` | Find files by name pattern | Your file explorer's search bar |

## Core Capabilities

### read_file — Looking at Things

The most common operation. You ask Amplifier about a file, and behind the scenes it reads it:

```
> What's in src/config.py?

[Tool: read_file] src/config.py
     1	DATABASE_URL = "postgres://localhost/myapp"
     2	DEBUG = True
     3	MAX_RETRIES = 3
```

Output comes in `cat -n` format — line numbers on the left, content on the right. This matters when you later need to edit: the line numbers help you identify exactly what to change, but they're not part of the file content itself.

**Large files** don't need to be read all at once. Amplifier can request specific ranges with `offset` and `limit`, reading up to 2,000 lines per call. For a huge log file, it might grab just the last 50 lines instead of swallowing the whole thing.

**Directories** work too. Point `read_file` at a folder and you get a listing:

```
> Show me what's in the src/ directory

[Tool: read_file] src/
DIR: components/
DIR: utils/
FILE: main.py
FILE: config.py
```

**Bundle resources** use `@bundle:path` notation to read files from installed bundles:

```
> Show me the recipe examples

[Tool: read_file] @recipes:examples/
DIR: workflows/
FILE: code-review.yaml
FILE: deploy.yaml
```

### write_file — Creating New Things

When you need an entirely new file, `write_file` creates it:

```
> Create a new utility module for date formatting

[Tool: write_file] src/utils/dates.py
Created src/utils/dates.py (24 lines)
```

There's one important safety rule: **if the file already exists, Amplifier must read it first.** This prevents accidentally blowing away a file's contents. If you ask to modify an existing file, Amplifier will read it, understand what's there, then decide whether to use `write_file` (for a complete rewrite) or `edit_file` (for a surgical change).

`write_file` is the right choice when you're creating something from scratch — a new module, a config file, a test fixture. It's the wrong choice for changing one line in an existing file.

### edit_file — Surgical Changes

Most of the time, you want to change *part* of a file while leaving everything else untouched. That's `edit_file` — it finds an exact string in the file and replaces it:

```
> Change DEBUG to False in the config

[Tool: read_file] src/config.py
[Tool: edit_file] src/config.py
  old: DEBUG = True
  new: DEBUG = False
```

The `old_string` must be **unique** in the file. If `return x` appears on five different lines, the edit will fail — you need to include enough surrounding context to pinpoint the exact location:

```
> Fix the calculate function to handle negative numbers

[Tool: edit_file] src/math.py
  old: def calculate(x):
           return x * 2
  new: def calculate(x):
           return abs(x) * 2
```

For renaming a variable everywhere in a file, use **replace_all**:

```
> Rename userName to user_name throughout src/auth.py

[Tool: edit_file] src/auth.py (replace_all)
  old: userName
  new: user_name
  Replaced 12 occurrences
```

### apply_patch — Multi-File Refactors

When a change spans multiple files — renaming a module, restructuring a package, or applying a large refactor — `apply_patch` handles it in one sweep. It supports three operations: `create_file`, `update_file`, and `delete_file`.

```
> Rename the auth module to authentication and update all imports

[Tool: apply_patch] create_file src/authentication.py
[Tool: apply_patch] update_file src/main.py (update imports)
[Tool: apply_patch] update_file src/api.py (update imports)
[Tool: apply_patch] delete_file src/auth.py
```

This is the tool Amplifier reaches for when a change touches many files at once. It's especially useful for refactors where consistency across files matters.

### glob — Finding Files

Before you can read or edit a file, you need to know where it is. `glob` finds files by pattern:

```
> Find all the Python test files in this project

[Tool: glob] **/test_*.py
  tests/test_auth.py
  tests/test_api.py
  tests/unit/test_models.py
  tests/unit/test_utils.py
  (4 files, sorted by modification time)
```

Results come back sorted by modification time (newest first), which is useful — the file you just changed is probably the one you care about. Common non-source directories like `node_modules`, `.git`, `.venv`, and `__pycache__` are automatically excluded.

Common patterns:

```
**/*.py              # All Python files, recursively
src/**/*.{js,ts}     # JS and TS files under src/
**/Dockerfile        # All Dockerfiles
**/*.test.js         # All JS test files
```

## Practical Examples

### Exploring an Unfamiliar Project

```
> I just cloned this repo. What's the structure?

[Tool: read_file] ./
[Tool: glob] **/*.py
[Tool: read_file] README.md

The project has 3 Python packages under src/, with tests in tests/.
The main entry point is src/main.py...
```

Amplifier often reads multiple files in parallel when exploring — it's faster than reading them one at a time.

### Fixing a Bug in Existing Code

```
> The retry logic in api_client.py isn't respecting the max retries setting

[Tool: read_file] src/api_client.py
[Tool: edit_file] src/api_client.py
  old: for i in range(3):
  new: for i in range(self.max_retries):
```

A single `edit_file` call — precise, minimal, and safe. The rest of the file stays exactly as it was.

### Creating a New Feature File

```
> Add a caching layer for the API responses

[Tool: glob] src/**/*.py (to understand the project layout)
[Tool: read_file] src/api_client.py (to understand the interface)
[Tool: write_file] src/cache.py (create the new module)
[Tool: edit_file] src/api_client.py (wire in the cache import)
```

Notice the pattern: glob to orient, read to understand, write to create, edit to integrate.

## Which Tool Should I Use?

When you're asking Amplifier to make changes, it follows this decision flow:

```
Need to find files?
  --> glob

Need to look at a file or directory?
  --> read_file

Need to create a brand new file?
  --> write_file

Need to change part of an existing file?
  --> edit_file

Need to change multiple files at once?
  --> apply_patch

Need to completely rewrite an existing file?
  --> read_file first, then write_file
```

In practice, most editing work uses `edit_file` because most changes are surgical. `apply_patch` shows up during refactors. `write_file` is mainly for new files. And `read_file` is everywhere — it's the most-used tool by far.

## Tips

**Always read before editing.** Both `write_file` and `edit_file` require that the file has been read first in the conversation. This isn't just a rule — it means Amplifier understands what's in the file before changing it.

**Include context for uniqueness.** If an `edit_file` fails because the string isn't unique, include the surrounding lines. Instead of matching just `return x`, match the whole function signature and body.

**Use replace_all for renames.** When you need to rename a variable, function, or string across a file, `replace_all` is cleaner than multiple individual edits. For cross-file renames, consider the LSP `rename` operation instead.

**Parallel reads are fast.** When Amplifier needs to understand multiple files, it reads them all at once rather than sequentially. You'll see this when exploring codebases or preparing for multi-file changes.

**Mind the indentation.** When `read_file` shows output, the line numbers are prefixed but the actual content starts after a tab. When specifying `old_string` for edits, use the file's real indentation — never include the line number prefix.

**Bundle paths work everywhere.** Any tool that takes a file path accepts the `@bundle:path` format — `@recipes:examples/review.yaml` reads from the recipes bundle, just as you'd expect.

## Next Steps

- Learn about [Search Tools](./search.md) for finding content *inside* files with `grep`
- See [Bash Tool](./bash.md) for running shell commands alongside file operations
- Explore [LSP (Code Intelligence)](./lsp.md) for semantic operations like cross-file rename
