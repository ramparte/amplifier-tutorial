---
id: modules
type: concepts
title: "Understanding Modules"
---

# Understanding Modules

Modules are the fundamental building blocks of Amplifier. They represent the "bricks and studs" philosophy—self-contained units with standardized interfaces that snap together to create powerful AI applications. Understanding modules is essential to mastering Amplifier's architecture.

## What is a Module?

A module is a **self-contained unit of functionality** that:

- Has a **single, well-defined purpose**
- Implements a **specific contract** (interface)
- Can be **swapped, upgraded, or replaced** without affecting others
- Operates **independently** while integrating seamlessly

Think of modules like LEGO bricks. Each brick has a specific shape and purpose, but they all share the same connection system (studs and sockets). You can build anything by combining bricks, and you can always swap one brick for another of the same type.

### The Module Philosophy

```
┌─────────────────────────────────────────────────────────────┐
│                     YOUR APPLICATION                        │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐        │
│  │Provider │  │  Tool   │  │ Context │  │  Hook   │        │
│  │ Module  │  │ Module  │  │ Module  │  │ Module  │        │
│  └────┬────┘  └────┬────┘  └────┬────┘  └────┬────┘        │
│       │            │            │            │              │
│       └────────────┴─────┬──────┴────────────┘              │
│                          │                                  │
│                   ┌──────┴──────┐                           │
│                   │Orchestrator │                           │
│                   │   Module    │                           │
│                   └─────────────┘                           │
└─────────────────────────────────────────────────────────────┘
```

Modules communicate through **contracts**, not implementations. This means:

- A Claude provider and an OpenAI provider are interchangeable
- A filesystem tool and a database tool both implement the same tool contract
- You can add observability hooks without changing any other code

## The 5 Module Types

Amplifier defines exactly five module types. Each has a specific role and contract:

| Type | Purpose | Contract | When It Runs |
|------|---------|----------|--------------|
| **Provider** | LLM backends | `complete()` | When AI response needed |
| **Tool** | Agent capabilities | `execute()` | When LLM requests action |
| **Orchestrator** | Main engine | The agent loop | Continuously during session |
| **Context** | Memory & state | `messages` | Before each LLM call |
| **Hook** | Observers | `events` | On system events |

### Provider Modules

Providers connect Amplifier to language models. They abstract away the differences between OpenAI, Anthropic, Azure, local models, and others.

**Contract:** `complete(messages, tools, config) -> response`

```python
# All providers implement the same interface
class Provider(Protocol):
    async def complete(
        self,
        messages: list[Message],
        tools: list[Tool],
        config: CompletionConfig
    ) -> CompletionResponse:
        ...
```

**Key characteristics:**
- Handle authentication and API specifics internally
- Normalize responses to a common format
- Support streaming where available
- Manage rate limiting and retries

**Examples:** `AnthropicProvider`, `OpenAIProvider`, `AzureProvider`, `OllamaProvider`

### Tool Modules

Tools give agents the ability to act in the world. They represent capabilities the LLM can invoke to accomplish tasks.

**Contract:** `execute(arguments) -> result`

```python
class Tool(Protocol):
    name: str
    description: str
    parameters: JSONSchema
    
    async def execute(self, arguments: dict) -> ToolResult:
        ...
```

**Key characteristics:**
- Self-describing (name, description, parameters)
- Executed when the LLM decides to use them
- Return structured results back to the conversation
- Can be dynamically enabled/disabled

**Examples:** `read_file`, `write_file`, `bash`, `web_search`, `grep`

### Orchestrator Modules

The orchestrator is the brain of the agent. It runs the main loop: getting context, calling the LLM, executing tools, and repeating.

**Contract:** The agent loop itself

```python
class Orchestrator(Protocol):
    async def run(self, session: Session) -> None:
        while not done:
            # 1. Gather context
            messages = await context.get_messages()
            
            # 2. Call LLM
            response = await provider.complete(messages, tools)
            
            # 3. Process response
            if response.has_tool_calls:
                results = await execute_tools(response.tool_calls)
            
            # 4. Check completion
            done = response.is_final
```

**Key characteristics:**
- Controls the agent's decision-making flow
- Manages turn-taking between user and agent
- Coordinates all other module types
- Handles completion conditions

**Examples:** `StandardOrchestrator`, `ReActOrchestrator`, `PlanningOrchestrator`

### Context Modules

Context modules manage what the agent "knows" during a conversation. They build the message history and inject relevant information.

**Contract:** `messages` - provides the conversation context

```python
class Context(Protocol):
    async def get_messages(self, session: Session) -> list[Message]:
        ...
    
    async def add_message(self, message: Message) -> None:
        ...
```

**Key characteristics:**
- Maintain conversation history
- Inject system prompts and instructions
- Can add retrieved documents (RAG)
- Handle context window limits

**Examples:** `ConversationContext`, `SlidingWindowContext`, `RAGContext`

### Hook Modules

Hooks observe and react to system events without modifying the core flow. They're perfect for logging, analytics, guardrails, and side effects.

**Contract:** `events` - receives notifications

```python
class Hook(Protocol):
    async def on_event(self, event: Event) -> None:
        ...
```

**Key characteristics:**
- Passive observers (don't change main flow)
- Subscribe to specific event types
- Run asynchronously when events occur
- Can emit their own events

**Examples:** `LoggingHook`, `MetricsHook`, `GuardrailHook`, `AuditHook`

## Tool vs Hook

This distinction confuses many newcomers. Here's the key difference:

| Aspect | Tool | Hook |
|--------|------|------|
| **Triggered by** | LLM decides to use it | Code events fire |
| **Purpose** | Give agent capabilities | Observe/react to events |
| **Control flow** | Blocks until complete | Runs asynchronously |
| **Visibility** | LLM knows about it | LLM doesn't know |
| **Examples** | Read file, search web | Log events, track metrics |

### When to Use Tools

Use a tool when the **LLM needs to decide** whether to use the capability:

```python
# Tool: LLM chooses when to search
@tool
async def web_search(query: str) -> str:
    """Search the web for information."""
    return await search_engine.query(query)
```

The LLM sees this tool, understands its purpose, and invokes it when relevant.

### When to Use Hooks

Use a hook when you want to **observe without the LLM knowing**:

```python
# Hook: Automatically logs all tool calls
class ToolLoggingHook(Hook):
    async def on_event(self, event: Event):
        if event.type == "tool_call":
            logger.info(f"Tool called: {event.tool_name}")
```

The LLM never knows this hook exists—it just observes silently.

### The Decision Tree

```
Should the LLM decide when this runs?
    │
    ├── YES → Use a TOOL
    │         (agent capability)
    │
    └── NO → Use a HOOK
             (system observation)
```

## Module Composition

The power of modules comes from composition. You combine simple modules to create sophisticated behavior:

```yaml
# A simple but powerful agent configuration
provider: anthropic
orchestrator: standard

tools:
  - read_file
  - write_file
  - bash
  - web_search

context:
  - conversation
  - system_prompt

hooks:
  - logging
  - metrics
  - guardrails
```

### Adding New Capabilities

Want to add a new capability? Just add a module:

```yaml
tools:
  - read_file
  - write_file
  - bash
  - web_search
  - database_query  # New capability!
```

### Swapping Implementations

Need to change providers? Just swap:

```yaml
# Before
provider: openai

# After
provider: anthropic
```

Everything else stays the same.

## Module Contracts

Every module type has a **contract**—a promise about how it behaves. Contracts enable:

1. **Interchangeability**: Any module implementing the contract works
2. **Testing**: Mock modules that follow the contract
3. **Evolution**: Improve internals without breaking interfaces

### The Contract Principle

```
┌─────────────┐         ┌─────────────┐
│  Module A   │         │  Module B   │
│             │         │             │
│ (OpenAI)    │         │ (Anthropic) │
└──────┬──────┘         └──────┬──────┘
       │                       │
       └───────────┬───────────┘
                   │
            ┌──────┴──────┐
            │  Contract   │
            │ complete()  │
            └─────────────┘
                   │
            ┌──────┴──────┐
            │   Kernel    │
            └─────────────┘
```

The kernel only knows about contracts, not implementations. This is what makes modules truly interchangeable.

## Key Takeaways

1. **Five module types**: Provider, Tool, Orchestrator, Context, Hook—each with a specific purpose

2. **Contracts over implementations**: Modules are interchangeable because they implement standard contracts

3. **Tools vs Hooks**: Tools are LLM-invoked capabilities; hooks are silent observers

4. **Composition is key**: Combine simple modules to create powerful agents

5. **Single responsibility**: Each module does one thing well

6. **The LEGO principle**: Standard interfaces let you build anything from simple, swappable pieces

## What's Next?

Now that you understand modules, explore each type in depth:

- [Providers: Connecting to LLMs](./providers.md)
- [Tools: Agent Capabilities](./tools.md)
- [Orchestrators: The Agent Loop](./orchestrators.md)
- [Context: Memory and State](./context.md)
- [Hooks: Observability](./hooks.md)

Or see modules in action:

- [Building Your First Agent](../guides/first-agent.md)
- [Module Configuration](../guides/configuration.md)
