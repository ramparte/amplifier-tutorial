---
id: architecture
type: concepts
title: "Architecture Overview"
---

# Architecture Overview

This document provides a comprehensive overview of the system architecture, explaining how
components interact to create a flexible, extensible AI agent runtime.

## The Kernel Philosophy

The kernel follows a fundamental principle: **mechanism, not policy**.

This means the kernel provides the *how* but never the *what*. It offers primitives for:

- Session management and lifecycle
- Event emission and subscription
- Module mounting and discovery
- Tool invocation and result handling
- Hook execution chains

What it explicitly avoids:

- Deciding which tools to expose
- Defining conversation strategies
- Implementing specific behaviors
- Making business logic decisions

This separation keeps the kernel minimal and stable while allowing infinite customization
through modules. The kernel changes rarely; modules change constantly.

## Session Lifecycle

Every interaction flows through a well-defined session lifecycle:

```
User Input
    ↓
┌─────────────────────────────────────────────────────────────────┐
│                          SESSION                                 │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │ State: user_id, session_id, context, history            │    │
│  └─────────────────────────────────────────────────────────┘    │
└────────────────────────────┬────────────────────────────────────┘
                             ↓
┌─────────────────────────────────────────────────────────────────┐
│                        COORDINATOR                               │
│  • Validates input                                               │
│  • Manages conversation flow                                     │
│  • Handles turn transitions                                      │
└────────────────────────────┬────────────────────────────────────┘
                             ↓
┌─────────────────────────────────────────────────────────────────┐
│                       ORCHESTRATOR                               │
│  • Manages the agent loop                                        │
│  • Dispatches to provider                                        │
│  • Processes tool calls                                          │
│  • Executes hooks at each phase                                  │
└────────────────────────────┬────────────────────────────────────┘
                             ↓
              ┌──────────────┴──────────────┐
              ↓                             ↓
┌─────────────────────────┐   ┌─────────────────────────┐
│        PROVIDER         │   │      TOOLS/HOOKS        │
│  (Claude, GPT, etc.)    │   │  (Mounted Modules)      │
└─────────────────────────┘   └─────────────────────────┘
```

### Session States

A session progresses through these states:

1. **INITIALIZING** - Session created, loading configuration
2. **READY** - All modules mounted, waiting for input
3. **PROCESSING** - Actively handling a user turn
4. **AWAITING_TOOL** - Waiting for tool execution results
5. **COMPLETED** - Session ended normally
6. **ERROR** - Session ended due to an error

### The Agent Loop

The orchestrator runs the core agent loop:

```
┌──────────────────────────────────────────────────────────────┐
│                      AGENT LOOP                               │
│                                                               │
│   ┌─────────┐    ┌──────────┐    ┌─────────────┐            │
│   │  User   │───→│ Provider │───→│ Tool Calls? │            │
│   │  Input  │    │  (LLM)   │    └──────┬──────┘            │
│   └─────────┘    └──────────┘           │                    │
│                                   Yes   │   No               │
│                       ┌─────────────────┼──────────┐         │
│                       ↓                              ↓        │
│              ┌────────────────┐              ┌───────────┐   │
│              │ Execute Tools  │              │  Return   │   │
│              │ (with hooks)   │              │ Response  │   │
│              └───────┬────────┘              └───────────┘   │
│                      │                                       │
│                      └──────→ Back to Provider ─────────────┘│
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

The loop continues until the provider returns a response without tool calls,
or a maximum iteration limit is reached.

## Event Flow

The system uses an event-driven architecture for observability and extensibility.
Events flow through a central bus, allowing modules to subscribe to what they need.

### Event Categories

| Category | Purpose | Examples |
|----------|---------|----------|
| `session.*` | Session lifecycle | `session.started`, `session.ended` |
| `turn.*` | Turn processing | `turn.started`, `turn.completed` |
| `tool.*` | Tool invocation | `tool.called`, `tool.result` |
| `provider.*` | LLM interactions | `provider.request`, `provider.response` |
| `hook.*` | Hook execution | `hook.before`, `hook.after` |

### Event Structure

Every event follows a consistent structure:

```python
{
    "event_type": "tool.called",
    "timestamp": "2024-01-15T10:30:00Z",
    "session_id": "abc123",
    "payload": {
        "tool_name": "read_file",
        "arguments": {"path": "/src/main.py"},
        "invocation_id": "inv_456"
    },
    "metadata": {
        "source_module": "tool-filesystem",
        "turn_number": 3
    }
}
```

### Subscribing to Events

Modules subscribe to events using pattern matching:

```python
# Subscribe to all tool events
@event_handler("tool.*")
async def on_tool_event(event):
    logger.info(f"Tool event: {event.event_type}")

# Subscribe to specific event
@event_handler("session.started")
async def on_session_start(event):
    await initialize_user_context(event.payload.user_id)
```

## Module Mounting

Modules are the primary extension mechanism. They provide tools, hooks, context, and
behaviors that customize the system.

### Module Types

| Type | Purpose | Examples |
|------|---------|----------|
| **Tool Modules** | Provide callable tools | `tool-filesystem`, `tool-web` |
| **Hook Modules** | Intercept/modify flows | `hook-auth`, `hook-logging` |
| **Context Modules** | Inject system prompts | `context-project`, `context-user` |
| **Provider Modules** | LLM integrations | `provider-anthropic`, `provider-openai` |
| **Behavior Modules** | Complex behaviors | `behavior-agents`, `behavior-memory` |

### Mount Order

Modules are mounted in a specific order:

1. **Core modules** - Kernel-provided fundamentals
2. **Provider modules** - LLM connectivity
3. **Context modules** - System prompt building
4. **Tool modules** - Available capabilities
5. **Hook modules** - Processing interceptors
6. **Behavior modules** - High-level behaviors

Within each category, modules mount in the order specified in configuration.

### Module Resolution

When mounting, the system follows this flow:

```
┌────────────┐    ┌────────────┐    ┌────────────┐
│  Discover  │───→│  Validate  │───→│ Initialize │
└────────────┘    └────────────┘    └────────────┘
                                          │
                                          ↓
┌────────────┐    ┌────────────┐    ┌────────────┐
│  Activate  │←───│  Register  │←───│   Ready    │
└────────────┘    └────────────┘    └────────────┘
```

1. **Discovers** - Finds the module by name/path
2. **Validates** - Checks dependencies and compatibility
3. **Initializes** - Calls the module's init function
4. **Registers** - Adds tools/hooks to the session
5. **Activates** - Marks the module as ready

### Module Dependencies

Modules can declare dependencies on other modules:

```yaml
# module.yaml
name: my-tool
version: 1.0.0
dependencies:
  - tool-filesystem: ">=1.0.0"
  - hook-auth: ">=2.0.0"
```

The mount system ensures dependencies are satisfied before loading.

### Tool Registration

When a tool module mounts, it registers its tools:

```python
class FilesystemModule:
    def register_tools(self, registry):
        registry.add_tool(
            name="read_file",
            description="Read contents of a file",
            parameters=ReadFileParams,
            handler=self.read_file
        )
```

## Hook Execution

Hooks allow modules to intercept and modify the processing flow at defined points.

### Hook Points

| Hook Point | When | Use Cases |
|------------|------|-----------|
| `pre_turn` | Before processing starts | Auth, rate limiting |
| `pre_provider` | Before LLM call | Prompt modification |
| `post_provider` | After LLM response | Response filtering |
| `pre_tool` | Before tool execution | Permission checks |
| `post_tool` | After tool execution | Result transformation |
| `post_turn` | After turn completes | Logging, cleanup |

### Hook Chain

Multiple hooks at the same point form a chain:

```
Input → Hook1 → Hook2 → Hook3 → Continue Processing
              ↑
        (can modify or abort)
```

Hooks execute in registration order. Any hook can:
- **Pass through** - Let the chain continue
- **Modify** - Change the data and continue
- **Abort** - Stop the chain and return early

### Hook Example

```python
@hook("pre_tool")
async def validate_file_access(context, tool_name, arguments):
    if tool_name == "write_file":
        path = arguments.get("path", "")
        if not is_allowed_path(path):
            raise PermissionError(f"Cannot write to: {path}")
    return arguments  # Continue with (possibly modified) args
```

## Key Takeaways

Understanding the architecture helps you work effectively with the system:

### 1. Kernel is Minimal

The kernel provides mechanisms, not policies. It's deliberately simple and stable.
Customize through modules, not kernel changes.

### 2. Sessions are Stateful

Each session maintains its own state, mounted modules, and conversation history.
Sessions are isolated from each other.

### 3. Events Enable Observability

Everything emits events. Use them for logging, debugging, metrics, and extending
behavior without modifying core code.

### 4. Modules are the Extension Point

Need new functionality? Create a module. Need to modify behavior? Create a hook.
The module system is designed for this.

### 5. Order Matters

Mount order, hook order, and event order all affect behavior. Be intentional
about ordering in your configurations.

### 6. The Agent Loop is Central

Understanding the orchestrator's agent loop is key to understanding how
conversations flow and how tools integrate.

## Next Steps

With this architectural understanding, explore:

- **[Modules Deep Dive](./modules.md)** - Creating and configuring modules
- **[Hooks Guide](./hooks.md)** - Building effective hooks
- **[Bundles Guide](../bundles/index.md)** - Bundle composition patterns
- **[Advanced Topics](../advanced/index.md)** - Custom modules and tools
