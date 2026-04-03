---
id: writing-modules
type: contributing
title: "Writing a Module"
---

# Writing a Module

Every capability in Amplifier is delivered through a module. When you build a new tool, provider, hook, orchestrator, or context manager, you're extending the ecosystem in a way that any bundle can compose. This page walks you through creating a custom module from scratch — the contracts, the directory structure, testing, and publishing.

## What You'll Learn

By the end of this page, you'll know how to:

- Choose the right module type for your use case
- Implement the `mount()` function and satisfy protocol compliance
- Structure your module directory
- Test your module in isolation
- Package and publish it for others to use

---

## Module Types and Their Contracts

Amplifier defines exactly five module types. Each has a protocol — a set of methods your module must implement. Amplifier uses Python's structural typing (duck typing via protocols), so you never inherit from a base class. If your object has the right methods, it's a valid module.

| Type | Contract | Who Triggers It |
|------|----------|-----------------|
| **Provider** | `complete(request) → response` | Orchestrator calls it |
| **Tool** | `name`, `description`, `execute(input) → ToolResult` | LLM decides when |
| **Hook** | `__call__(event, data) → HookResult` | Kernel fires automatically |
| **Orchestrator** | Controls the request/response loop | Kernel runs it |
| **Context** | `add_message()`, `get_messages()`, `set_messages()`, `clear()` | Orchestrator and kernel |

The key design question: **"Should the LLM decide when this runs?"**

- **Yes** → Build a **Tool**. The LLM sees its name and description and chooses when to invoke it.
- **No** → Build a **Hook**. It fires automatically on lifecycle events, invisible to the LLM.
- **Neither** → It's probably a Provider, Orchestrator, or Context module.

---

## The mount() Function

Every module has one entry point: `mount()`. This is the function the kernel calls when loading your module. It receives a coordinator (for registering capabilities) and a config dict, and it returns a cleanup function.

```python
async def mount(coordinator, config):
    # 1. Create your module instance
    my_tool = MyTool(config)

    # 2. Register it with the coordinator
    await coordinator.mount("tools", my_tool, name="my-tool")

    # 3. Return a cleanup function
    async def cleanup():
        await my_tool.shutdown()
    return cleanup
```

Three things happen in `mount()`:

1. **Instantiate.** Create your module with whatever config it needs.
2. **Register.** Tell the coordinator what slot to put it in — `"tools"`, `"hooks"`, `"providers"`, etc.
3. **Cleanup.** Return an async function the kernel calls when the session ends.

The kernel validates protocol compliance at mount time. If your tool is missing `execute()`, or your provider doesn't implement `complete()`, the kernel rejects it immediately — not at runtime when a user hits the bug. Fail fast, fail loud.

---

## Example: Building a Custom Tool

Let's build a tool that counts lines in files — simple enough to show every piece of the pattern.

```python
# amplifier_module_tool_line_counter/tool.py
import os
from amplifier_core.models import ToolResult

class LineCounter:
    """Count lines in files within the workspace."""

    @property
    def name(self) -> str:
        return "line_counter"

    @property
    def description(self) -> str:
        return "Count the number of lines in one or more files"

    @property
    def parameters(self) -> dict:
        return {
            "type": "object",
            "properties": {
                "path": {
                    "type": "string",
                    "description": "File or directory path to count lines in"
                }
            },
            "required": ["path"]
        }

    async def execute(self, input: dict) -> ToolResult:
        path = input.get("path", ".")
        if not os.path.exists(path):
            return ToolResult(output=f"Error: path '{path}' does not exist")

        if os.path.isfile(path):
            count = sum(1 for _ in open(path, errors="replace"))
            return ToolResult(output=f"{path}: {count} lines")

        # Directory: count all files recursively
        total, results = 0, []
        for root, _, files in os.walk(path):
            for fname in sorted(files):
                fpath = os.path.join(root, fname)
                try:
                    count = sum(1 for _ in open(fpath, errors="replace"))
                except OSError:
                    count = 0
                results.append(f"  {fpath}: {count}")
                total += count
        return ToolResult(output=f"{chr(10).join(results)}\n\nTotal: {total} lines")
```

The `name`, `description`, and `parameters` properties tell the LLM what this tool does. `execute()` does the work. Errors return messages in `ToolResult` rather than raising exceptions — a module should never crash the kernel. The mount function wires it up:

```python
# amplifier_module_tool_line_counter/__init__.py
from .tool import LineCounter

async def mount(coordinator, config):
    await coordinator.mount("tools", LineCounter(), name="line_counter")
    async def cleanup():
        pass
    return cleanup
```

Once mounted, the LLM invokes your tool when it's relevant:

> How many lines of code are in the src directory?

```
[Tool: line_counter] src/
  src/auth.py: 142
  src/config.py: 67
  src/main.py: 203
  src/utils.py: 89

Total: 501 lines
```

---

## Module Directory Structure

A well-organized module follows this layout:

```
amplifier-module-tool-line-counter/
├── amplifier_module_tool_line_counter/
│   ├── __init__.py        # mount() function lives here
│   └── tool.py            # Tool implementation
├── tests/
│   ├── __init__.py
│   └── test_tool.py       # Unit tests
├── pyproject.toml          # Package metadata + entry point
├── README.md               # What this module does
└── LICENSE
```

The naming convention: the repository is `amplifier-module-{type}-{name}`, and the Python package uses underscores: `amplifier_module_{type}_{name}`.

---

## Testing Modules in Isolation

Modules are plain Python objects with known interfaces. You don't need a running Amplifier session to test them:

```python
# tests/test_tool.py
import pytest, tempfile, os
from amplifier_module_tool_line_counter.tool import LineCounter

@pytest.fixture
def tool():
    return LineCounter()

@pytest.fixture
def temp_file():
    with tempfile.NamedTemporaryFile(mode="w", suffix=".py", delete=False) as f:
        f.write("line one\nline two\nline three\n")
        path = f.name
    yield path
    os.unlink(path)

def test_name(tool):
    assert tool.name == "line_counter"

def test_description(tool):
    assert "lines" in tool.description.lower()

@pytest.mark.asyncio
async def test_count_file(tool, temp_file):
    result = await tool.execute({"path": temp_file})
    assert "3 lines" in result.output

@pytest.mark.asyncio
async def test_missing_path(tool):
    result = await tool.execute({"path": "/nonexistent/path"})
    assert "Error" in result.output
```

> Run the tests

```
[Tool: bash] pytest tests/ --verbose
===== test session starts =====
tests/test_tool.py::test_name PASSED
tests/test_tool.py::test_description PASSED
tests/test_tool.py::test_count_file PASSED
tests/test_tool.py::test_missing_path PASSED
===== 4 passed in 0.12s =====
```

Test the contract surface first — `name`, `description`, `parameters` — then test behavior. This ensures protocol compliance before you ever mount the module in a live session.

---

## Publishing Your Module

### pyproject.toml

The entry point registration is what makes your module discoverable by the kernel:

```toml
[project]
name = "amplifier-module-tool-line-counter"
version = "0.1.0"
description = "Count lines in files for Amplifier"
requires-python = ">=3.11"
dependencies = ["amplifier-core"]

[project.entry-points."amplifier.modules"]
tool-line-counter = "amplifier_module_tool_line_counter:mount"

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"
```

The entry point key (`tool-line-counter`) is the module ID that bundles reference. The value points Python to your `mount()` function.

### Using Your Module in a Bundle

Once published to a git repository, any bundle can reference it:

```yaml
tools:
  - module: tool-line-counter
    source: git+https://github.com/yourname/amplifier-module-tool-line-counter@main
```

### Adding to the Ecosystem Catalog

When your module is ready for the community, submit a PR to the main Amplifier repository adding your entry to `docs/MODULES.md`:

```markdown
| **tool-line-counter** | Count lines in files | @yourname | [repo](https://github.com/yourname/amplifier-module-tool-line-counter) |
```

---

## Key Takeaways

1. **Every module starts with `mount()`.** The kernel calls it, you register your capabilities, you return cleanup.

2. **Protocols, not inheritance.** Implement the right methods and your class is a valid module. No base classes, no decorators, no framework magic.

3. **The LLM triggers tools; code triggers hooks.** This is the fundamental design decision when choosing your module type.

4. **Test in isolation.** Modules are plain Python objects. Test them with pytest without running Amplifier.

5. **Entry points make it discoverable.** The `[project.entry-points."amplifier.modules"]` section in `pyproject.toml` is how the kernel finds your module.

6. **Error handling is non-negotiable.** Return errors in your results. Never raise unhandled exceptions. Never crash the kernel.

7. **Name it right.** `amplifier-module-{type}-{name}` for the repo, `amplifier_module_{type}_{name}` for the package. Consistency enables discovery.
