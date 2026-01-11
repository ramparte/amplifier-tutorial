---
id: hooks
type: concepts
title: "Understanding Hooks"
---

# Understanding Hooks

Hooks are Amplifier's mechanism for observing and reacting to events in the agent lifecycle. They allow you to extend agent behavior without modifying core logic, enabling cross-cutting concerns like logging, metrics, guardrails, and custom integrations.

## What is a Hook?

A hook is a **lifecycle observer** that receives notifications when specific events occur during agent execution. Unlike tools that agents actively invoke, hooks passively listen and respond to the natural flow of agent activity.

Think of hooks as event listeners attached to the agent runtime. When an agent starts a turn, calls a tool, receives a response, or completes its work, hooks are notified and can take action.

```python
from amplifier_core.hooks import Hook, HookContext

class LoggingHook(Hook):
    """A simple hook that logs agent activity."""
    
    async def on_turn_start(self, ctx: HookContext) -> None:
        print(f"Turn started: {ctx.turn_id}")
    
    async def on_turn_end(self, ctx: HookContext) -> None:
        print(f"Turn completed in {ctx.duration_ms}ms")
```

Hooks are registered with the agent runtime and automatically invoked at the appropriate lifecycle points. You don't call hooks directly—the system calls them for you.

## Hook vs Tool

Understanding the distinction between hooks and tools is fundamental to Amplifier's design:

| Aspect | Tool | Hook |
|--------|------|------|
| **Invocation** | Agent explicitly calls | System automatically triggers |
| **Control** | Agent decides when to use | Fires on lifecycle events |
| **Visibility** | Agent sees and chooses | Transparent to agent |
| **Purpose** | Provide capabilities | Observe and react |
| **Return value** | Returns data to agent | No return to agent |

**Tools are capabilities**—they extend what an agent *can do*. A file-reading tool gives the agent the ability to read files. The agent decides when and how to use it.

**Hooks are observers**—they extend what *happens* during execution. A logging hook records every tool call, but the agent doesn't know or care that logging is happening.

This separation keeps agent logic clean while allowing powerful cross-cutting functionality.

## Event Types

Hooks can subscribe to various lifecycle events. Here are the primary event categories:

### Session Events

- `on_session_start` — A new session begins
- `on_session_end` — A session completes or terminates
- `on_session_resume` — A persisted session resumes

### Turn Events

- `on_turn_start` — An agent turn begins (user message received)
- `on_turn_end` — An agent turn completes (response generated)
- `on_turn_error` — An error occurs during a turn

### Tool Events

- `on_tool_call_start` — Agent initiates a tool call
- `on_tool_call_end` — Tool execution completes
- `on_tool_call_error` — Tool execution fails

### Message Events

- `on_user_message` — User submits a message
- `on_assistant_message` — Agent generates a response
- `on_system_message` — System injects context

### Provider Events

- `on_llm_request` — Request sent to LLM provider
- `on_llm_response` — Response received from provider
- `on_llm_stream_chunk` — Streaming chunk received

Each event provides a `HookContext` with relevant data about the event, including timing, identifiers, and payload information.

## Built-in Hooks

Amplifier provides several hooks out of the box:

### MetricsHook

Collects timing and usage metrics for observability:

```python
from amplifier_core.hooks import MetricsHook

metrics_hook = MetricsHook(
    emit_interval=60,  # Emit aggregated metrics every 60 seconds
    include_tool_timing=True
)
```

### GuardrailHook

Enforces safety constraints and policies:

```python
from amplifier_core.hooks import GuardrailHook

guardrail_hook = GuardrailHook(
    block_patterns=[r"password", r"secret"],
    max_tool_calls_per_turn=50
)
```

### AuditHook

Records a complete audit trail of agent activity:

```python
from amplifier_core.hooks import AuditHook

audit_hook = AuditHook(
    output_path="./audit.jsonl",
    include_payloads=True
)
```

### ContextInjectionHook

Injects dynamic context into agent prompts:

```python
from amplifier_core.hooks import ContextInjectionHook

context_hook = ContextInjectionHook(
    injectors=[
        lambda ctx: f"Current time: {datetime.now()}",
        lambda ctx: f"Session ID: {ctx.session_id}"
    ]
)
```

## Creating Hooks

Creating custom hooks is straightforward. Implement the `Hook` base class and override the event methods you care about.

### Basic Hook Structure

```python
from amplifier_core.hooks import Hook, HookContext
from typing import Optional

class MyCustomHook(Hook):
    """Custom hook with selective event handling."""
    
    def __init__(self, config: dict):
        self.config = config
    
    async def on_turn_start(self, ctx: HookContext) -> None:
        # Called when a turn begins
        pass
    
    async def on_tool_call_end(self, ctx: HookContext) -> None:
        # Called after each tool completes
        tool_name = ctx.tool_name
        duration = ctx.duration_ms
        print(f"Tool {tool_name} completed in {duration}ms")
```

### Hook with State

Hooks can maintain state across events:

```python
class TokenCounterHook(Hook):
    """Tracks token usage across a session."""
    
    def __init__(self):
        self.total_input_tokens = 0
        self.total_output_tokens = 0
    
    async def on_llm_response(self, ctx: HookContext) -> None:
        usage = ctx.payload.get("usage", {})
        self.total_input_tokens += usage.get("input_tokens", 0)
        self.total_output_tokens += usage.get("output_tokens", 0)
    
    async def on_session_end(self, ctx: HookContext) -> None:
        print(f"Session totals: {self.total_input_tokens} in, "
              f"{self.total_output_tokens} out")
```

### Registering Hooks

Register hooks when creating an agent or session:

```python
from amplifier_core import Agent

agent = Agent(
    name="my-agent",
    hooks=[
        MetricsHook(),
        MyCustomHook(config={"verbose": True}),
        TokenCounterHook()
    ]
)
```

### Hook Ordering

Hooks execute in registration order. If order matters (e.g., a guardrail should run before logging), register them accordingly:

```python
hooks=[
    GuardrailHook(),   # Check constraints first
    AuditHook(),       # Then record the activity
    MetricsHook()      # Finally collect metrics
]
```

### Error Handling in Hooks

Hook errors are isolated—a failing hook won't crash the agent:

```python
class SafeHook(Hook):
    async def on_turn_end(self, ctx: HookContext) -> None:
        try:
            await self.do_something_risky()
        except Exception as e:
            # Log but don't propagate
            logger.error(f"Hook error: {e}")
```

## Key Takeaways

1. **Hooks are observers, not actors.** They watch lifecycle events without interfering with agent logic. The agent doesn't know hooks exist.

2. **Use hooks for cross-cutting concerns.** Logging, metrics, auditing, guardrails, and context injection are perfect hook use cases.

3. **Tools vs hooks is about control.** Agents control tool usage; the system controls hook invocation. Choose based on who should decide when the functionality runs.

4. **Hooks execute in order.** Registration order determines execution order. Place guardrails before audit hooks if you want to audit blocked requests.

5. **Keep hooks lightweight.** Hooks run synchronously in the event flow. Heavy processing should be offloaded to background tasks.

6. **Hooks are isolated.** A failing hook won't crash your agent, but you should still handle errors gracefully.

7. **State is per-instance.** Each hook instance maintains its own state. Create new instances for independent tracking.

Hooks embody Amplifier's philosophy of composable, non-intrusive extensibility. They let you add powerful functionality to any agent without touching its core implementation.
