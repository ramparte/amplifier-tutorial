---
id: hooks
type: concepts
title: "Understanding Hooks"
---

# Understanding Hooks

Hooks are the observation layer of the Amplifier ecosystem. They allow you to monitor, react to, and extend agent behavior without modifying core logic. Think of hooks as event listeners that tap into the agent lifecycle at specific points.

## What is a Hook?

A hook is a lifecycle observer that receives notifications about events occurring during agent execution. Unlike tools that perform actions, hooks observe and react to what's already happening.

```python
from amplifier import Hook, HookContext

class LoggingHook(Hook):
    """Simple hook that logs all events."""
    
    async def on_event(self, ctx: HookContext) -> None:
        print(f"[{ctx.event_type}] {ctx.timestamp}: {ctx.data}")
```

Hooks are passive by design. They receive event data, can perform side effects (logging, metrics, notifications), but they don't modify the agent's execution flow. This separation keeps your agent logic clean while enabling rich observability.

### Key Characteristics

- **Non-blocking**: Hooks should complete quickly to avoid slowing the agent
- **Read-only context**: Hooks observe events but don't modify them
- **Fail-safe**: Hook failures don't crash the agent
- **Ordered execution**: Multiple hooks run in registration order

## Hook vs Tool

Understanding the difference between hooks and tools is fundamental:

| Aspect | Hook | Tool |
|--------|------|------|
| **Purpose** | Observe and react | Perform actions |
| **Invocation** | Automatic on events | Explicit by agent |
| **Modifies state** | No (observation only) | Yes (side effects expected) |
| **Blocking** | Should be fast | Can be long-running |
| **Failure impact** | Logged, continues | Reported to agent |

**Use a hook when you want to:**
- Log or audit agent activity
- Collect metrics and telemetry
- Send notifications on specific events
- Validate or monitor behavior patterns

**Use a tool when you need to:**
- Perform an action the agent requests
- Return data the agent will use
- Modify external state
- Execute long-running operations

## Event Types

Hooks subscribe to specific event types that occur during the agent lifecycle:

### Session Events

```python
class SessionEvents:
    SESSION_START = "session.start"      # Agent session begins
    SESSION_END = "session.end"          # Agent session completes
    SESSION_ERROR = "session.error"      # Unhandled error in session
```

### Message Events

```python
class MessageEvents:
    USER_MESSAGE = "message.user"        # User sends a message
    ASSISTANT_MESSAGE = "message.assistant"  # Agent responds
    SYSTEM_MESSAGE = "message.system"    # System injects context
```

### Tool Events

```python
class ToolEvents:
    TOOL_CALL = "tool.call"              # Agent invokes a tool
    TOOL_RESULT = "tool.result"          # Tool returns a result
    TOOL_ERROR = "tool.error"            # Tool execution fails
```

### LLM Events

```python
class LLMEvents:
    LLM_REQUEST = "llm.request"          # Request sent to LLM
    LLM_RESPONSE = "llm.response"        # Response received from LLM
    LLM_STREAM_CHUNK = "llm.stream"      # Streaming chunk received
```

## Built-in Hooks

Amplifier provides several hooks out of the box for common observability needs:

### ConsoleLoggingHook

Logs all events to the console with configurable verbosity:

```python
from amplifier.hooks import ConsoleLoggingHook

hook = ConsoleLoggingHook(
    level="INFO",
    include_events=["tool.call", "tool.result"]
)
```

### MetricsHook

Collects timing and count metrics for analysis:

```python
from amplifier.hooks import MetricsHook

hook = MetricsHook()

# After session, access metrics
print(hook.metrics.tool_call_count)
print(hook.metrics.avg_response_time)
```

### FileAuditHook

Writes a complete audit trail to a file:

```python
from amplifier.hooks import FileAuditHook

hook = FileAuditHook(
    path="./logs/session-{session_id}.jsonl",
    include_content=True  # Include full message content
)
```

### WebhookNotifierHook

Sends HTTP notifications on specific events:

```python
from amplifier.hooks import WebhookNotifierHook

hook = WebhookNotifierHook(
    url="https://api.example.com/events",
    events=["session.end", "tool.error"],
    headers={"Authorization": "Bearer token"}
)
```

## Creating Hooks

Building custom hooks is straightforward. Extend the base `Hook` class and implement the methods you need:

### Basic Hook Structure

```python
from amplifier import Hook, HookContext

class MyCustomHook(Hook):
    """A custom hook with selective event handling."""
    
    def __init__(self, config: dict = None):
        self.config = config or {}
        self.event_count = 0
    
    @property
    def subscribed_events(self) -> list[str]:
        """Specify which events this hook receives."""
        return ["tool.call", "tool.result", "session.end"]
    
    async def on_event(self, ctx: HookContext) -> None:
        """Handle incoming events."""
        self.event_count += 1
        
        if ctx.event_type == "tool.call":
            await self._handle_tool_call(ctx)
        elif ctx.event_type == "session.end":
            await self._handle_session_end(ctx)
    
    async def _handle_tool_call(self, ctx: HookContext) -> None:
        tool_name = ctx.data.get("tool_name")
        print(f"Tool called: {tool_name}")
    
    async def _handle_session_end(self, ctx: HookContext) -> None:
        print(f"Session ended. Total events observed: {self.event_count}")
```

### Registering Hooks

Register hooks when configuring your agent:

```python
from amplifier import Agent

agent = Agent(
    hooks=[
        ConsoleLoggingHook(),
        MyCustomHook({"verbose": True}),
        MetricsHook()
    ]
)
```

### Hook Context

The `HookContext` object provides rich information about each event:

```python
class HookContext:
    event_type: str          # The event type (e.g., "tool.call")
    timestamp: datetime      # When the event occurred
    session_id: str          # Current session identifier
    data: dict               # Event-specific payload
    metadata: dict           # Additional context (user_id, etc.)
```

### Best Practices

1. **Keep hooks fast**: Offload heavy work to background tasks
2. **Handle errors gracefully**: Don't let hook failures affect the agent
3. **Be selective**: Subscribe only to events you need
4. **Use async patterns**: Leverage async for I/O operations

```python
async def on_event(self, ctx: HookContext) -> None:
    try:
        # Quick validation
        if not self._should_process(ctx):
            return
        
        # Offload heavy work
        asyncio.create_task(self._process_async(ctx))
    except Exception as e:
        # Log but don't raise
        logger.warning(f"Hook error: {e}")
```

## Key Takeaways

1. **Hooks are observers**: They watch the agent lifecycle without modifying it. Use them for logging, metrics, notifications, and auditing.

2. **Hooks complement tools**: While tools perform actions for the agent, hooks monitor what's happening. Both are essential for production systems.

3. **Event types are granular**: Subscribe to specific events like `tool.call` or `session.end` rather than receiving everything.

4. **Built-in hooks cover common cases**: Use `ConsoleLoggingHook`, `MetricsHook`, and `FileAuditHook` before building custom solutions.

5. **Custom hooks are simple**: Extend `Hook`, specify `subscribed_events`, and implement `on_event`. Keep them fast and error-tolerant.

6. **Observability enables reliability**: Hooks provide the visibility needed to debug, optimize, and trust your agent systems in production.

Hooks transform opaque agent execution into transparent, observable systems. Start with built-in hooks to understand behavior, then create custom hooks as your monitoring needs evolve.
