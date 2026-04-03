---
id: hooks
type: concept
title: "Hooks"
---

# Hooks

Tools wait to be called. Hooks don't wait for anyone. Every time something
happens in an Amplifier session — a tool fires, a message arrives, a turn
begins — hooks are already there, watching. They log, they validate, they
inject context, they ask for permission. And the LLM never even knows they
exist.

## What is a Hook?

A hook is a **lifecycle observer** — a function that fires automatically at
specific points during agent execution. The crucial word is *automatically*.
The LLM doesn't choose to invoke a hook the way it chooses to call a tool.
Hooks are **code-decided**, triggered by events in the system, not by the
model's reasoning.

This is the fundamental distinction in Amplifier's module system:

| | Tool | Hook |
|---|------|------|
| **Triggered by** | The LLM decides | Code fires automatically |
| **Visible to LLM?** | Yes — sees name, description, schema | No — completely invisible |
| **Returns to LLM?** | Yes — results enter conversation | Only if the hook injects context |
| **Example** | `bash` — LLM decides to run a command | `hooks-logging` — every event logged to disk |

A `read_file` tool makes sense because the agent should decide when it needs
to see a file. But a logging hook makes sense because you want every event
logged whether the agent "wants" that or not.

## How It Works

### The Hook Contract

Every hook follows the same contract — receive an event, return a result:

```python
async def my_hook(event: str, data: dict[str, Any]) -> HookResult:
    # Observe, validate, or react to the event
    return HookResult(action="continue")
```

That's it. The kernel calls your function with an event name and a data dict.
You inspect what happened and return a `HookResult` telling the kernel what
to do next.

### HookResult Actions

The `HookResult` is where hooks get their power. You're not limited to
passive observation — you can actively shape what happens:

| Action | What It Does | Example |
|--------|-------------|---------|
| `continue` | Proceed normally | Default — nothing to report |
| `deny` | Block the operation | Security violation, validation failure |
| `modify` | Transform the event data | Redact PII, enrich context |
| `inject_context` | Add a message to the agent's conversation | Linter feedback, status updates |
| `ask_user` | Pause and request human approval | Sensitive file writes, production deploys |

**Action precedence matters.** When multiple hooks fire on the same event,
blocking actions (`deny`, `ask_user`) always win over non-blocking ones
(`inject_context`, `modify`, `continue`). A security hook's `deny` can never
be silently overridden by an informational hook's `continue`.

### Lifecycle Events

Hooks tap into events that span the full session lifecycle:

| Event | When It Fires |
|-------|--------------|
| `session_start` | A new session begins |
| `session_end` | The session wraps up |
| `turn_start` | The agent begins processing a user message |
| `turn_end` | The agent finishes its response |
| `tool:pre` | Just before a tool executes |
| `tool:post` | Right after a tool returns |
| `provider_request` | An LLM API call is about to go out |
| `provider_response` | An LLM response has arrived |

Each event delivers relevant data. A `tool:pre` event includes the tool
name, input parameters, and file paths. A `provider_response` event includes
token counts and timing. Your hook decides which events it cares about and
ignores the rest.

## Using Hooks

Hooks are configured in your bundle YAML, just like tools and providers.
You declare them, the kernel mounts them, and they start firing automatically.

```yaml
bundle:
  name: my-assistant
  version: 1.0.0

hooks:
  - module: hooks-logging
  - module: hooks-streaming-ui
  - module: hooks-status-context
  - module: hooks-approval
    config:
      require_approval:
        - bash
        - write_file
```

That's a bundle with four hooks. The logging hook records every event. The
streaming UI hook renders real-time output. The status context hook injects
git status and environment info. The approval hook pauses before `bash` and
`write_file` calls to ask for permission. None of these require the LLM to
do anything — they just work.

### Hooks in the Ecosystem

The Amplifier ecosystem includes a rich set of hooks for common needs:

| Hook | What It Does |
|------|-------------|
| **hooks-logging** | Writes every event to a JSONL log file — your audit trail |
| **hooks-streaming-ui** | Renders real-time streaming display as the LLM generates tokens |
| **hooks-status-context** | Injects git status, environment info, and workspace context into each turn |
| **hooks-approval** | Requires user confirmation before sensitive tool calls proceed |
| **hooks-redaction** | Strips PII (emails, tokens, keys) from data before it reaches the LLM |
| **hooks-todo-reminder** | Surfaces task tracking reminders so the agent stays on track |
| **hooks-skills-visibility** | Shows available skills to the agent before each request |
| **hooks-python-check** | Auto-runs linting and type checking after Python file edits |

Notice the range. Some hooks are pure observation (logging). Some inject
information (status-context, skills-visibility, todo-reminder). Some gate
operations (approval). Some transform data (redaction). The hook contract is
flexible enough to support all of these patterns.

## Creating Your Own

Let's build a practical hook from scratch — a simple event logger that
writes tool calls to a file.

### A Simple Logging Hook

```python
import json
from datetime import datetime, timezone
from pathlib import Path
from amplifier_core.models import HookResult

LOG_PATH = Path("./logs/tool-calls.jsonl")

async def tool_logger(event: str, data: dict) -> HookResult:
    """Log every tool call with timestamp and parameters."""
    if event != "tool:pre":
        return HookResult(action="continue")

    LOG_PATH.parent.mkdir(parents=True, exist_ok=True)

    entry = {
        "timestamp": datetime.now(timezone.utc).isoformat(),
        "tool": data.get("tool_name"),
        "input": data.get("tool_input"),
    }

    with open(LOG_PATH, "a") as f:
        f.write(json.dumps(entry) + "\n")

    return HookResult(action="continue")
```

Register it as a module with a `mount` function:

```python
async def mount(coordinator, config):
    registry = coordinator.hook_registry
    unregister = registry.register(
        event="tool:pre",
        handler=tool_logger,
        priority=0,
        name="tool_logger"
    )

    async def cleanup():
        unregister()
    return cleanup
```

And declare it in `pyproject.toml`:

```toml
[project.entry-points."amplifier.modules"]
hooks-my-logger = "my_hooks.tool_logger:mount"
```

That's a complete hook module. The kernel discovers it by entry point, calls
`mount()`, and your logger starts receiving events.

### The Approval Gate Pattern

The approval hook is one of the most powerful patterns. It intercepts tool
calls and pauses execution until a human says "yes":

```python
SENSITIVE_TOOLS = {"bash", "write_file", "delete_file"}

async def approval_gate(event: str, data: dict) -> HookResult:
    """Require user approval for sensitive operations."""
    tool_name = data.get("tool_name", "")

    if tool_name not in SENSITIVE_TOOLS:
        return HookResult(action="continue")

    file_path = data.get("tool_input", {}).get("file_path", "")
    command = data.get("tool_input", {}).get("command", "")
    detail = file_path or command or "(no details)"

    return HookResult(
        action="ask_user",
        approval_prompt=f"Allow {tool_name}: {detail}?",
        approval_options=["Allow once", "Allow always", "Deny"],
        approval_timeout=300.0,
        approval_default="deny",
    )
```

When this hook fires on a `tool:pre` event for `bash`, the session pauses.
The user sees the prompt, makes a choice, and the session continues or
blocks accordingly. The LLM never knows any of this happened — it just sees
the tool result (or a denial message).

### Hooks That Use Models

Here's a subtle point: hooks can use LLM calls internally. A redaction hook
might call a small model to classify whether text contains PII. A
summarization hook might condense long tool output before it enters context.

The key difference from tools remains the **triggering mechanism**. The outer
LLM never decides to invoke the hook — code fires it automatically. What the
hook does internally (including calling models) is its own business.

```python
async def smart_redaction(event: str, data: dict) -> HookResult:
    """Use a fast model to classify and redact sensitive content."""
    content = data.get("tool_result", {}).get("output", "")

    # Call a small, fast model for classification
    classification = await classify_pii(content, model="fast")

    if classification.has_pii:
        redacted = redact_matches(content, classification.matches)
        return HookResult(
            action="modify",
            data={**data, "tool_result": {"output": redacted}},
            user_message="Redacted sensitive content from output",
            user_message_level="info",
        )

    return HookResult(action="continue")
```

The agent sees clean output. It never knows redaction happened. That's the
hook philosophy — invisible infrastructure that keeps things safe.

## Best Practices

**Keep hooks fast.** Pre-tool hooks (`tool:pre`) block execution until they
return. Heavy work should be offloaded to background tasks or deferred to
post-event hooks. Use async I/O for external calls.

**Fail open by default.** A crashing hook should not take down the session.
Catch exceptions and return `HookResult(action="continue")` unless you
explicitly intend to block. The kernel treats hook failures as non-fatal, but
your hook should handle errors gracefully too.

```python
async def safe_hook(event: str, data: dict) -> HookResult:
    try:
        # Your logic here
        return do_validation(data)
    except Exception as e:
        logger.warning(f"Hook error: {e}")
        return HookResult(action="continue")
```

**Be selective about events.** Subscribe only to the events your hook needs.
A logging hook might want everything, but an approval hook should fire only
on `tool:pre` for specific tools. Unnecessary hook invocations add latency.

**Use `deny` sparingly.** Blocking is powerful but disruptive. Prefer
`ask_user` over `deny` when the situation is ambiguous — let the human
decide. Reserve `deny` for clear violations (invalid paths, known-bad
patterns).

**Mind the injection budget.** Context injections consume tokens. Amplifier
enforces a configurable budget (default 10,000 tokens per turn). Keep
injections concise and use `ephemeral: True` for transient state like todo
reminders that refresh every turn.

**Use priorities to control ordering.** Hooks fire in priority order (lower
number = earlier). Put security hooks at low priorities (0-10) so they run
first, and informational hooks at higher priorities (50+).

## Key Takeaways

1. **Hooks are code-decided, not LLM-decided.** They fire automatically on
   lifecycle events. The LLM never chooses to invoke a hook — that's what
   makes them different from tools.

2. **The contract is simple.** Receive `(event, data)`, return a
   `HookResult`. Five actions cover the full range: `continue`, `deny`,
   `modify`, `inject_context`, and `ask_user`.

3. **Hooks shape behavior invisibly.** Logging, approval gates, PII
   redaction, context injection — hooks provide safety and observability
   without the agent ever knowing they're there.

4. **The ecosystem is rich.** Eight built-in hooks cover logging, streaming,
   status context, approval, redaction, reminders, skill visibility, and
   code checking. Start there before building custom.

5. **Approval gates are the power pattern.** Intercepting `tool:pre` events
   and returning `ask_user` gives humans a safety valve over sensitive
   operations without disrupting the agent's flow.

6. **Hooks can use models internally.** A hook can call an LLM for
   classification or summarization. The distinction isn't *what* hooks do —
   it's *who decides when* they run.

---

## Related Concepts

- [Modules](./modules.md) — The five module types and how hooks fit into the architecture
- [Bundles](./bundles.md) — Packaging hooks with tools, agents, and configuration
- [Architecture](./architecture.md) — How hooks integrate with the kernel event system
