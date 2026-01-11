---
id: how-its-different
type: quickstart
title: "How It's Different"
---

# How Amplifier is Different

Amplifier represents a fundamentally different approach to building AI-powered applications.
Rather than treating AI as a black box to be wrapped in layers of abstraction, Amplifier
embraces simplicity, modularity, and developer control.

## Comparison Overview

Most AI frameworks fall into two categories: either they're too opinionated and lock you
into specific patterns, or they're too low-level and require you to build everything from
scratch. Amplifier charts a middle path.

| Aspect | Traditional Frameworks | Amplifier |
|--------|----------------------|-----------|
| Architecture | Monolithic, tightly coupled | Modular, composable |
| Extensibility | Plugin systems, inheritance | First-class module contracts |
| Multi-Agent | Bolted on, complex | Native, simple |
| Customization | Configuration files | Behavior composition |
| Learning Curve | Steep, framework-specific | Gradual, principle-based |

### What We're Not

Amplifier is not:

- A wrapper around a single LLM provider
- An opinionated application framework
- A no-code/low-code platform
- A managed service with vendor lock-in

### What We Are

Amplifier is:

- A kernel for AI agent orchestration
- A collection of composable behaviors
- A philosophy of building with AI
- An open platform for innovation

## Modular Architecture

At the heart of Amplifier is the "bricks and studs" philosophy. Like interlocking building
blocks, every component has well-defined connection points that allow them to snap together
predictably.

### The Kernel

The Amplifier kernel is intentionally minimal. It provides:

- **Session lifecycle management** - Creating, persisting, and resuming conversations
- **Module orchestration** - Loading and coordinating modules at runtime
- **Event dispatch** - A simple pub/sub system for cross-cutting concerns
- **Provider abstraction** - A thin layer over LLM providers

What it doesn't do is equally important. The kernel has no opinions about:

- How you structure your prompts
- What tools you expose to agents
- How agents communicate with each other
- Where you store your data

### Modules Over Plugins

Traditional plugin systems are often afterthoughts. They expose limited hooks and require
you to work around the framework's assumptions. Amplifier modules are different.

Every capability in Amplifier is a module:

```
amplifier-core/
  modules/
    providers/     # LLM provider implementations
    tools/         # Tool definitions and handlers
    hooks/         # Lifecycle event handlers
    context/       # Context providers
```

Modules follow explicit contracts. A provider module must implement the Provider protocol.
A tool module must implement the Tool protocol. There's no magic, no hidden behavior.

### Thin Bundles

Bundles are curated collections of modules composed for specific use cases. The key insight
is that bundles should be thin - they compose behaviors rather than implementing them.

```yaml
# A bundle is mostly composition
name: code-assistant
extends: foundation
modules:
  - tool-bash
  - tool-file-ops
  - tool-lsp
behaviors:
  - code-review
  - test-runner
```

This approach means you can understand exactly what a bundle does by reading its manifest.
There's no hidden complexity buried in base classes or inherited configurations.

## Extensibility

Extensibility in Amplifier isn't a feature - it's the foundation.

### Protocol-Based Design

Every extension point is defined by a protocol (interface). Want to add a new LLM provider?
Implement the Provider protocol:

```python
class Provider(Protocol):
    async def generate(self, messages: list[Message]) -> Response: ...
    async def stream(self, messages: list[Message]) -> AsyncIterator[Chunk]: ...
```

Want to add a new tool? Implement the Tool protocol:

```python
class Tool(Protocol):
    name: str
    description: str
    parameters: JsonSchema
    async def execute(self, params: dict) -> ToolResult: ...
```

The protocols are minimal and stable. They define the contract, not the implementation.

### Hooks for Cross-Cutting Concerns

Need to log every tool call? Add a hook. Need to filter responses? Add a hook. Need to
inject context? Add a hook.

```python
@hook("tool.before_call")
async def log_tool_calls(event: ToolCallEvent):
    logger.info(f"Calling {event.tool_name} with {event.arguments}")
```

Hooks are non-invasive. They observe and optionally modify behavior without requiring
changes to the core system.

### No Framework Lock-In

Amplifier doesn't require you to adopt it entirely. You can:

- Use just the kernel for agent orchestration
- Use individual modules in your existing application
- Gradually adopt more capabilities as needed
- Replace any component with your own implementation

## Multi-Agent

Multi-agent systems are notoriously complex. Amplifier makes them simple by design.

### Agents Are Sessions

In Amplifier, an agent is just a session with a specific configuration. There's no special
"Agent" class or complex initialization:

```python
# Create an agent by creating a session with behaviors
research_agent = await amplifier.session(
    behaviors=["web-research", "summarization"]
)

# Create another agent with different capabilities
code_agent = await amplifier.session(
    behaviors=["code-analysis", "refactoring"]
)
```

### Simple Delegation

Agents delegate to other agents through the task tool. The pattern is straightforward:

1. Parent agent decides to delegate
2. Child agent receives instruction and context
3. Child agent works independently
4. Child agent returns a result
5. Parent agent continues

There's no complex message-passing infrastructure, no shared state to synchronize, no
race conditions to handle. Each agent session is independent.

### Parallel Execution

Because agents are independent sessions, parallel execution is natural:

```python
# Launch multiple agents in parallel
results = await asyncio.gather(
    research_agent.run("Find information about X"),
    code_agent.run("Analyze the authentication module"),
)
```

### Hierarchical Composition

Complex workflows emerge from simple compositions:

```
main-agent
├── planning-agent
│   └── estimation-agent
├── implementation-agent
│   ├── frontend-agent
│   └── backend-agent
└── review-agent
```

Each level only knows about its immediate children. The planning agent doesn't know or
care how the implementation agent structures its work internally.

## Open Source

Amplifier is fully open source under a permissive license. But open source means more
than just "code is available."

### Transparent Development

All development happens in the open. Issues, discussions, and roadmap decisions are
visible and participatory. There are no private feature branches or surprise releases.

### Community-Driven Evolution

The module system means the community can extend Amplifier without waiting for core
changes. New providers, tools, and behaviors can be developed and shared independently.

### No Artificial Limitations

There are no "enterprise" features held back, no usage limits, no telemetry requirements.
You get the complete system, and you can deploy it however you want.

### Sustainable Ecosystem

The modular architecture means:

- Core remains stable and maintained
- Community modules can evolve independently
- Breaking changes are isolated to specific modules
- You can lock versions of individual components

## The Amplifier Philosophy

Beyond technical differences, Amplifier embodies a philosophy:

1. **Simplicity over features** - We'd rather do fewer things well
2. **Composition over inheritance** - Build by combining, not extending
3. **Explicit over implicit** - No magic, no hidden behavior
4. **Trust the developer** - You know your domain better than we do
5. **Embrace AI as a partner** - AI builds the code, humans guide the vision

This philosophy permeates every design decision. When evaluating new features, we ask:
Does this make things simpler? Does it compose well? Is it explicit? Does it trust
developers?

If the answer is no, we find a different approach or don't add the feature at all.

## Next Steps

Ready to experience the difference?

- [Installation Guide](./installation.md) - Get Amplifier running in minutes
- [Your First Agent](./first-agent.md) - Build something real
- [Core Concepts](../concepts/overview.md) - Understand the foundations
- [Module Development](../guides/creating-modules.md) - Extend Amplifier yourself
