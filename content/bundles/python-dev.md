---
id: python-dev
type: bundle
title: "Python Dev Bundle"
---

# Python Dev Bundle

## What is the Python Dev Bundle?

You've used grep to find text and LSP to navigate definitions. But Python development isn't just reading code — it's a tight loop of writing, checking, and fixing. You write a function, and you need to know immediately: does it lint? Does it type-check? Did you break something downstream?

The Python Dev bundle integrates code quality and code intelligence into a single package. It supersedes the older LSP Python bundle by combining two capabilities that used to live apart: a `python_check` tool that runs formatting, linting, and type checking in one pass, and a `code-intel` agent that handles complex semantic navigation. On top of that, it hooks into file writes — every time you save a Python file, checks run automatically, so problems surface before they compound.

The result is a development loop where code quality isn't a separate step you remember to run. It's built into the way you work.

## What's Included

The bundle provides two main capabilities and an automatic checking hook:

| Component | What It Does |
|-----------|-------------|
| **`python_check` tool** | Runs ruff (formatting + linting), pyright (type checking), and stub detection on files or inline code |
| **`python-dev:code-intel` agent** | LSP/Pyright specialist for multi-step semantic code navigation |
| **Auto-check hook** | Fires on every Python file write — runs lint and type checks automatically |

### python_check Tool

The `python_check` tool is your single entry point for code quality. It orchestrates three checkers behind the scenes:

- **Ruff** — formatting and linting (replaces black, isort, flake8, and dozens of other tools)
- **Pyright** — static type checking with full type inference
- **Stub detection** — identifies missing type stubs for third-party packages

You can run it against files, directories, or raw code strings, and optionally auto-fix what's fixable.

### code-intel Agent

The `python-dev:code-intel` agent is a Pyright/LSP specialist. It handles complex, multi-step code navigation that would require chaining several LSP operations together — tracing inheritance hierarchies, mapping module dependencies, building call graphs, debugging type flows across files. Think of it as the senior developer who already knows how to use all the LSP operations and will chain them together intelligently for you.

## Getting Started

### Prerequisites

The bundle requires **ruff** and **pyright** installed as Python packages in your environment — not just on PATH:

```bash
pip install ruff pyright
```

Both must be importable from the Python environment your project uses. If you're working inside a virtual environment, install them there.

### Your First Check

Run a check against a file:

> Check src/main.py for issues

```
[Tool: python_check]
  paths: ["src/main.py"]

✓ Format: clean
✗ Lint: 2 issues
  src/main.py:14:5  F841  Local variable 'result' is assigned but never used
  src/main.py:27:1  E302  Expected 2 blank lines before function definition
✗ Types: 1 error
  src/main.py:31:12  error  Argument of type "str" cannot be assigned
                            to parameter "count" of type "int"

Summary: 2 lint issues, 1 type error
```

Three checkers, one tool call. The response tells you exactly what to fix and where.

### Auto-Fix

Many lint and format issues can be fixed automatically:

> Fix the formatting and lint issues in src/

```
[Tool: python_check]
  paths: ["src/"]
  fix: true

Fixed 14 issues across 6 files:
  src/main.py       — 2 fixes (unused import, blank lines)
  src/utils.py      — 5 fixes (import sorting, trailing whitespace)
  src/models.py     — 3 fixes (formatting)
  src/api/views.py  — 4 fixes (line length, blank lines)
```

Auto-fix handles formatting and safe lint fixes. Type errors require manual intervention — the tool reports them, you fix the logic.

## Key Features

### Automatic Checking on File Writes

This is the feature that changes your workflow. Every time a Python file is written — whether you write it, an agent writes it, or a recipe step writes it — the auto-check hook fires. It runs lint and type checks and reports problems immediately.

> Add a caching decorator to src/utils.py

```
[Tool: edit_file] src/utils.py — added cache_result decorator

[Hook: python-dev auto-check]
  src/utils.py — lint clean, types clean ✓
```

If the edit introduces problems, you see them instantly:

> Refactor the database connection to use async

```
[Tool: edit_file] src/db.py — converted to async/await

[Hook: python-dev auto-check]
  src/db.py:42:5  error  "await" not allowed in non-async function
  src/db.py:58:12 error  Return type "Coroutine[Any, Any, Connection]"
                         is not assignable to "Connection"
```

No separate "now run the linter" step. The feedback loop is immediate.

### Checking Inline Code

You can check code without writing it to a file first — useful for validating a snippet before committing to it:

> Check this code for issues

```
[Tool: python_check]
  content: |
    def process(items: list[str]) -> int:
        for item in items:
            result = item.upper()
        return result

  ✗ Lint: 1 issue
    line 3:9  F841  Local variable 'result' assigned in loop body
  ✗ Types: 1 error
    line 4:12  error  "result" is possibly unbound
```

### Selective Checks

Run only the checks you need with the `checks` parameter:

> Just run type checking on src/api/

```
[Tool: python_check]
  paths: ["src/api/"]
  checks: ["types"]

  src/api/views.py:22:8  error  Cannot access attribute "user_id"
                                for class "AnonymousUser"
```

Available check types: `format`, `lint`, `types`, `stubs`.

### Using the code-intel Agent

For simple, single-step LSP operations — go to definition, find references, hover — use the LSP tool directly. For complex tasks that require chaining multiple operations, delegate to the code-intel agent:

> Trace how request authentication works from the middleware through to the permission check

```
[Agent: python-dev:code-intel]
  1. Found AuthMiddleware.process_request in src/middleware/auth.py:18
  2. Traces to authenticate_token() in src/auth/tokens.py:42
     → Returns AuthResult with user and permissions
  3. AuthResult flows to PermissionCheck.has_access() in src/permissions.py:65
     → Checks user.permissions against required_perms
  4. Permission failure raises PermissionDenied (src/exceptions.py:12)

  Call chain: process_request → authenticate_token → has_access
  Key types: AuthResult(user: User, permissions: Set[str])
```

**When to delegate vs. use LSP directly:**

| Task | Approach |
|------|----------|
| "Where is `UserModel` defined?" | LSP `goToDefinition` directly |
| "What type does `get_config()` return?" | LSP `hover` directly |
| "Find all callers of `validate()`" | LSP `incomingCalls` directly |
| "Trace the data flow from API to database" | Delegate to `code-intel` |
| "Map the inheritance hierarchy of all handlers" | Delegate to `code-intel` |
| "Why is this type mismatch happening across modules?" | Delegate to `code-intel` |

The rule of thumb: if you'd need to chain three or more LSP operations, or if the task requires judgment about which operations to run next, delegate.

## Configuration

The bundle reads configuration from your project's `pyproject.toml`:

```toml
[tool.amplifier-python-dev]
# Checks to run automatically on file writes (default: lint + types)
auto_checks = ["lint", "types"]

# Additional ruff rules to enable or disable
ruff_select = ["I", "UP"]
ruff_ignore = ["E501"]

# Pyright type checking mode
type_checking_mode = "basic"  # "off", "basic", "standard", "strict"
```

Ruff and Pyright also respect their own configuration sections in `pyproject.toml` — `[tool.ruff]` and `[tool.pyright]` — so your existing project settings carry over automatically.

## A Practical Workflow

Here's a real development loop using the Python Dev bundle — writing a new feature from scratch:

> Add a retry decorator with exponential backoff to src/utils/retry.py

```
[Tool: write_file] src/utils/retry.py — created with retry decorator

[Hook: python-dev auto-check]
  src/utils/retry.py:8:22  F821  Undefined name 'wraps'
  src/utils/retry.py:15:5  error  "sleep" is not defined
```

The auto-check caught two missing imports immediately. Fix them:

> Add the missing imports for wraps and sleep

```
[Tool: edit_file] src/utils/retry.py — added imports

[Hook: python-dev auto-check]
  src/utils/retry.py — lint clean, types clean ✓
```

Now verify the whole utils package is consistent:

> Run a full check on src/utils/

```
[Tool: python_check]
  paths: ["src/utils/"]

  ✓ Format: clean
  ✓ Lint: clean
  ✓ Types: clean
  ✓ Stubs: no missing stubs

  Summary: all checks passed
```

Clean across the board. The whole loop — write, catch, fix, verify — happened without leaving the conversation.

## Tips

- **Trust the auto-check.** It fires on every write. If you don't see warnings, the file is clean. You don't need to run `python_check` manually after every edit — only for broader sweeps across directories.

- **Use `fix: true` for formatting.** Don't manually fix import order or blank line issues. Let `python_check` handle them automatically. Save your attention for type errors that need actual thought.

- **Check content before files.** When prototyping a tricky function, pass it as `content` first. Once the snippet is clean, write it to disk.

- **Delegate complex navigation.** If you find yourself running three LSP operations in a row trying to trace something, stop and delegate to `code-intel`. It'll get there faster with better context.

- **Configure your project's pyproject.toml.** The bundle's auto-checks respect your project's ruff and pyright settings. Set them once and every check — manual or automatic — follows your rules.

## Next Steps

- Learn about the [LSP Tool](../tools/lsp.md) for direct code intelligence operations
- Explore the [Foundation Bundle](./foundation.md) for the core tools Python Dev builds on
- See [Recipes](./recipes.md) for automating multi-step Python workflows
- Read about [Skills](../skills/index.md) for adding Python domain knowledge
