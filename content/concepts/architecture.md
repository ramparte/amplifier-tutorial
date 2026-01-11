---
id: architecture
type: concepts
title: "Architecture Overview"
---

# Architecture Overview

This document provides a comprehensive overview of the Amplifier kernel architecture,
explaining the core design principles, component interactions, and data flow patterns
that make the system work.

## The Kernel Philosophy

**Mechanism, not policy.**

The Amplifier kernel follows a fundamental design principle borrowed from operating system
design: provide mechanisms, not policies. This means the kernel supplies the building
blocks and infrastructure for AI agent execution without dictating how those blocks
must be used.

### What This Means in Practice

The kernel provides:

- **Session management** - The mechanism for tracking conversation state
- **Provider abstraction** - The mechanism for communicating with LLMs
- **Tool execution** - The mechanism for agents to take actions
- **Event emission** - The mechanism for observability and extension
- **Module mounting** - The mechanism for adding capabilities

The kernel does NOT dictate:

- Which LLM provider you must use
- What tools are available to agents
- How conversation history is formatted
- What retry strategies to employ
- How errors should be presented to users

### The Litmus Test

When deciding whether something belongs in the kernel, ask:

1. **Is it a mechanism or a policy?** Mechanisms go in kernel; policies go in modules.
2. **Would removing it break the agent loop?** If yes, it's kernel. If no, it's a module.
3. **Does it need to work the same way everywhere?** Kernel components are universal.

Examples:
- Event emission → Kernel (mechanism for observability)
- Retry logic → Module (policy decision)
- Provider protocol → Kernel (mechanism for LLM communication)
- Rate limiting → Module (policy for resource management)

## Session Lifecycle

A session represents a complete conversation between a user and an AI agent. Understanding
the session lifecycle is essential for building applications with Amplifier.

### Component Flow

```
User Input
    ↓
┌─────────┐
│ Session │ ← Maintains state, history, configuration
└────┬────┘
     ↓
┌─────────────┐
│ Coordinator │ ← Manages session lifecycle, applies hooks
└──────┬──────┘
       ↓
┌─────────────┐
│ Orchestrator│ ←→ Provider (LLM API)
└──────┬──────┘
       ↓
   Tools/Hooks ← External capabilities and integrations
```

### Session States

A session progresses through distinct states:

| State | Description |
|-------|-------------|
| `created` | Session initialized but not yet started |
| `running` | Actively processing user input or LLM response |
| `idle` | Waiting for user input |
| `suspended` | Paused, can be resumed later |
| `completed` | Finished successfully |
| `failed` | Terminated due to error |

### The Coordinator

The Coordinator is the session's traffic controller. It:

1. **Receives user messages** and adds them to session history
2. **Invokes hooks** at appropriate lifecycle points
3. **Delegates to the Orchestrator** for LLM interaction
4. **Manages state transitions** between session states
5. **Handles interrupts** and graceful shutdown

### The Orchestrator

The Orchestrator manages the agent loop - the core cycle of:

```
┌─────────────────────────────────────────┐
│                                         │
│  ┌─────────┐    ┌────────┐    ┌─────┐  │
│  │ Prepare │ → │ Invoke │ → │Parse│  │
│  │ Request │    │Provider│    │Reply│  │
│  └─────────┘    └────────┘    └──┬──┘  │
│       ↑                          │      │
│       │    ┌─────────────┐       │      │
│       └────│Execute Tools│←──────┘      │
│            └─────────────┘              │
│                                         │
└─────────────────────────────────────────┘
```

The loop continues until:
- The LLM responds without requesting tool calls
- A maximum iteration limit is reached
- An error occurs or the session is interrupted

### Session Configuration

Sessions accept configuration that controls behavior:

```python
session = Session(
    provider=my_provider,           # Required: LLM provider
    system_prompt="You are...",     # Optional: Agent instructions
    tools=[tool1, tool2],           # Optional: Available tools
    hooks=[hook1, hook2],           # Optional: Lifecycle hooks
    max_iterations=50,              # Optional: Loop limit
    metadata={"user_id": "123"}     # Optional: Custom metadata
)
```

## Event Flow

Events provide observability into everything happening within a session. They enable
logging, debugging, metrics collection, and reactive integrations.

### Event Architecture

```
┌─────────────────────────────────────────────────┐
│                  Session                         │
│                                                  │
│  Component A ──emit──→ ┌─────────────┐          │
│                        │ Event Bus   │──→ Hook 1│
│  Component B ──emit──→ │             │──→ Hook 2│
│                        │ (in-process)│──→ Logger│
│  Component C ──emit──→ └─────────────┘          │
│                                                  │
└─────────────────────────────────────────────────┘
```

### Event Categories

Events are organized into categories:

| Category | Examples | Purpose |
|----------|----------|---------|
| `session` | `session.started`, `session.completed` | Lifecycle tracking |
| `message` | `message.user`, `message.assistant` | Conversation flow |
| `tool` | `tool.call.started`, `tool.call.completed` | Tool execution |
| `provider` | `provider.request`, `provider.response` | LLM interaction |
| `error` | `error.tool`, `error.provider` | Error tracking |

### Event Structure

Every event follows a consistent structure:

```python
{
    "type": "tool.call.completed",      # Event type identifier
    "session_id": "abc-123",            # Session context
    "timestamp": "2024-01-15T10:30:00Z",# When it occurred
    "data": {                           # Event-specific payload
        "tool_name": "read_file",
        "duration_ms": 45,
        "result": "..."
    }
}
```

### Subscribing to Events

Hooks can subscribe to specific event types:

```python
class MetricsHook(Hook):
    def __init__(self):
        self.subscriptions = [
            "tool.call.completed",
            "provider.response"
        ]
    
    async def on_event(self, event: Event):
        if event.type == "tool.call.completed":
            self.record_tool_latency(event.data)
```

### Event Ordering Guarantees

- Events are emitted synchronously during execution
- Events within a session are ordered by emission time
- No guarantees across sessions (they may run concurrently)

## Module Mounting

Modules extend the kernel's capabilities without modifying its core. The mounting
system provides a clean separation between kernel mechanisms and application-specific
functionality.

### Module Types

| Type | Purpose | Examples |
|------|---------|----------|
| **Provider** | LLM communication | Anthropic, OpenAI, Azure |
| **Tool** | Agent capabilities | File operations, web search |
| **Hook** | Lifecycle integration | Logging, metrics, guards |
| **Storage** | Persistence | Session storage, memory |

### The Mount Protocol

Modules implement a standard protocol for lifecycle management:

```python
class Module(Protocol):
    async def mount(self, context: MountContext) -> None:
        """Called when module is attached to a session."""
        pass
    
    async def unmount(self) -> None:
        """Called when module is detached or session ends."""
        pass
```

### Mount Context

When mounting, modules receive context about their environment:

```python
@dataclass
class MountContext:
    session_id: str           # The session being mounted to
    config: dict              # Module-specific configuration
    event_bus: EventBus       # For emitting/subscribing to events
    kernel_version: str       # For compatibility checks
```

### Module Dependencies

Modules can declare dependencies on other modules:

```python
class DatabaseTool(Tool):
    dependencies = ["connection-pool"]  # Requires connection pool module
    
    async def mount(self, context: MountContext):
        # Connection pool is guaranteed to be mounted first
        self.pool = context.get_module("connection-pool")
```

### Mount Order

The kernel mounts modules in dependency order:

1. Core kernel components initialize
2. Providers mount (LLM connectivity)
3. Storage modules mount (persistence ready)
4. Tools mount (capabilities available)
5. Hooks mount (observability active)

### Hot Reloading

Some module types support hot reloading during a session:

```python
# Add a tool mid-session
await session.mount_tool(new_tool)

# Remove a hook
await session.unmount_hook(old_hook)
```

Note: Providers and storage typically cannot be hot-reloaded as they maintain
critical state.

## Key Takeaways

### 1. Separation of Concerns

The kernel provides mechanisms; modules implement policies. This separation enables:
- Flexible application architecture
- Easy testing through module substitution
- Clear boundaries for debugging

### 2. Event-Driven Observability

Everything emits events. This means:
- Full visibility into system behavior
- Non-invasive monitoring and debugging
- Reactive integrations without modifying core logic

### 3. Composable by Design

The module system enables composition:
- Mix and match providers, tools, and hooks
- Build application-specific bundles
- Share and reuse modules across projects

### 4. Session-Centric Model

Sessions are the fundamental unit:
- All state lives within a session
- Sessions are independent and isolated
- Sessions can be persisted, resumed, and analyzed

### 5. Predictable Lifecycle

Clear state machines govern behavior:
- Session states are explicit and observable
- Mount/unmount lifecycle is deterministic
- The agent loop follows a consistent pattern

## What's Next

Now that you understand the architecture:

- **[Sessions Deep Dive](./sessions.md)** - Detailed session management
- **[Building Modules](../guides/building-modules.md)** - Create custom modules
- **[Event Reference](../reference/events.md)** - Complete event catalog
- **[Provider Protocol](../reference/provider-protocol.md)** - LLM integration details
