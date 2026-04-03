---
id: python-check-tool
type: tool
title: "Python Check Tool"
---

# Python Check Tool

## What Is python_check?

Every Python developer knows the routine: write code, run the linter, fix the warnings, run the type checker, fix those warnings, re-run everything to make sure you didn't introduce new problems. It's tedious when you do it yourself, and it's easy to skip steps.

The `python_check` tool collapses that entire routine into a single call. It orchestrates three checkers behind the scenes — **ruff** for formatting and linting, **pyright** for static type checking, and a **stub detector** for missing type stubs — and returns a unified report. One tool call, one response, everything you need to know about the quality of your Python code.

Better still, it runs automatically. Every time a Python file is written during your session, an auto-check hook fires and reports problems before you even think to ask. The manual tool is there for broader sweeps — checking a whole directory, running only specific checks, or auto-fixing what's fixable.

## Prerequisites

The tool requires **ruff** and **pyright** installed as Python packages in your environment:

```bash
pip install ruff pyright
```

Both must be importable via `sys.executable -m` — meaning they need to live in the same Python environment your project uses. If you're working inside a virtual environment, install them there, not globally.

## Core Capabilities

### Parameters

The tool takes four parameters, all optional:

- **paths** — a list of files or directories to check. `["src/main.py"]` checks one file; `["src/"]` checks an entire directory tree.
- **content** — a Python code string to check without writing it to a file first. Useful for validating a snippet before committing to it.
- **fix** — if `true`, auto-fix formatting and safe lint issues. Only works with `paths`, not `content`.
- **checks** — a list of specific checks to run: `"format"`, `"lint"`, `"types"`, `"stubs"`. Defaults to all four.

You must provide either `paths` or `content`, but not both.

### Output

Every response includes four fields:

- **success** — `true` if there are no errors (warnings are acceptable)
- **clean** — `true` if there are no issues at all, not even warnings
- **summary** — a human-readable summary of what was found
- **issues** — a list of individual problems, each with `file`, `line`, `code`, `message`, and `severity`

The distinction between `success` and `clean` matters. A file can succeed (no errors blocking correctness) while still not being clean (some style warnings remain). When you need a strict quality gate, check `clean`. For everyday work, `success` is what matters.

## Checking Files and Directories

The most common use: point it at code and see what's wrong.

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

Check an entire directory to sweep for problems across a package:

> Run a full check on the api module

```
[Tool: python_check]
  paths: ["src/api/"]

✓ Format: clean
✗ Lint: 5 issues
  src/api/views.py:12:1    E302  Expected 2 blank lines
  src/api/views.py:45:11   F401  'json' imported but unused
  src/api/auth.py:8:5      F841  Local variable 'token' assigned but never used
  src/api/auth.py:22:80    E501  Line too long (94 > 88)
  src/api/models.py:67:1   W291  Trailing whitespace
✗ Types: 2 errors
  src/api/views.py:22:8   error  Cannot access attribute "user_id"
                                 for class "AnonymousUser"
  src/api/auth.py:35:16   error  "verify_token" is not defined

Summary: 5 lint issues, 2 type errors
```

## Auto-Fixing Issues

Many lint and formatting problems have mechanical fixes — import sorting, blank lines, trailing whitespace. Don't fix these by hand:

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

Auto-fix handles formatting and safe lint corrections. Type errors are never auto-fixed — they require you to change the actual logic.

A practical pattern: fix first, then verify.

> Fix what you can, then show me what's left

```
[Tool: python_check]
  paths: ["src/"]
  fix: true

Fixed 8 issues across 3 files.

[Tool: python_check]
  paths: ["src/"]

✓ Format: clean
✓ Lint: clean
✗ Types: 1 error
  src/api/auth.py:35:16  error  "verify_token" is not defined

Summary: 1 type error remaining
```

The mechanical noise is gone. What's left is the real problem that needs thought.

## Checking Inline Code

You can validate a snippet before writing it to a file — useful when you're prototyping a tricky function:

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

Both the lint check and the type checker caught the same underlying bug from different angles: `result` is only defined inside the loop, so it's unbound if `items` is empty.

## Running Selective Checks

Sometimes you only care about one dimension of quality. Use the `checks` parameter to run a subset:

> Just run type checking on the api module

```
[Tool: python_check]
  paths: ["src/api/"]
  checks: ["types"]

  src/api/views.py:22:8  error  Cannot access attribute "user_id"
                                for class "AnonymousUser"
```

> Check only formatting across the whole project

```
[Tool: python_check]
  paths: ["src/"]
  checks: ["format"]

✗ Format: 3 files need reformatting
  src/utils.py
  src/api/views.py
  src/models.py
```

Available check types: `format`, `lint`, `types`, `stubs`.

## The Auto-Check Hook

The most important thing about `python_check` isn't the tool itself — it's the hook. Every time a Python file is written during your session, the auto-check fires automatically. It runs a fast subset of checks (lint and types by default) and injects any issues directly into the conversation context.

You don't call it. You don't configure it per-file. It just happens:

> Add a retry decorator to src/utils/retry.py

```
[Tool: write_file] src/utils/retry.py — created with retry decorator

[Hook: python-dev auto-check]
  src/utils/retry.py:8:22  F821  Undefined name 'wraps'
  src/utils/retry.py:15:5  error  "sleep" is not defined
```

The agent sees these issues immediately and can fix them in the same turn:

> (agent fixes the missing imports automatically)

```
[Tool: edit_file] src/utils/retry.py — added imports

[Hook: python-dev auto-check]
  src/utils/retry.py — lint clean, types clean ✓
```

This creates a tight write-check-fix loop that catches problems before they compound. By the time you're done writing code, it's already been checked.

## Configuration

The tool and hook respect configuration from your project's `pyproject.toml`:

```toml
[tool.amplifier-python-dev]
# Checks to run on auto-check (default: lint + types)
auto_checks = ["lint", "types"]

# Ruff rules to enable or disable
ruff_select = ["I", "UP"]
ruff_ignore = ["E501"]

# Pyright strictness level
type_checking_mode = "basic"  # "off", "basic", "standard", "strict"
```

Ruff and Pyright also respect their own standard configuration sections — `[tool.ruff]` and `[tool.pyright]` — so your existing project settings carry over automatically. You don't need to duplicate anything.

## A Practical Workflow

Here's the full loop: check a directory, see what's wrong, auto-fix the mechanical problems, then deal with the real issues.

> I want to clean up the utils package before the PR

```
[Tool: python_check]
  paths: ["src/utils/"]

✓ Format: clean
✗ Lint: 7 issues
✗ Types: 2 errors

Summary: 7 lint issues, 2 type errors across 4 files
```

> Fix what's auto-fixable

```
[Tool: python_check]
  paths: ["src/utils/"]
  fix: true

Fixed 5 issues across 3 files.
```

> What's left?

```
[Tool: python_check]
  paths: ["src/utils/"]

✓ Format: clean
✗ Lint: 2 issues
  src/utils/cache.py:18:5   F841  Local variable 'ttl' assigned but never used
  src/utils/retry.py:42:11  F811  Redefinition of unused 'logger'
✗ Types: 2 errors
  src/utils/cache.py:25:16   error  "expire" is not a known member of "Redis"
  src/utils/retry.py:30:12   error  Return type "None" not assignable to "int"

Summary: 2 lint issues, 2 type errors
```

Four issues that need real thought, not mechanical fixing. Now you can focus.

## Tips

**Let the hook do the work.** After every file write, the auto-check fires. If you don't see warnings, the file is clean. You only need to call `python_check` manually for broader sweeps across directories or when you want `fix: true`.

**Fix before you verify.** Run with `fix: true` first to clear the noise, then run again without it to see what's actually left. The mechanical fixes are never interesting — get them out of the way.

**Check `content` before writing files.** When prototyping something tricky, pass the code as a `content` string first. Once the snippet is clean, write it to disk. This avoids the write-check-fix-rewrite cycle.

**Use selective checks for speed.** If you only care about type safety right now, run `checks: ["types"]`. Skipping format and lint checks when they're not relevant saves time on large codebases.

**Configure `pyproject.toml` once.** Your ruff and pyright settings apply to every check — manual and automatic. Set your team's standards in the config and every check enforces them consistently.

## Next Steps

- Learn about the [Python Dev Bundle](../bundles/python-dev.md) for the full package including the code-intel agent
- See [LSP Tool](./lsp.md) for semantic code navigation — definitions, references, and call hierarchies
- Explore [Bash Tool](./bash.md) for running tests after your code passes all checks
- Read about [Filesystem Tools](./filesystem.md) for the file editing that triggers auto-checks
