---
id: what-is-amplifier
type: quickstart
title: "What is Amplifier?"
---

# What is Amplifier?

Amplifier is a modular AI agent framework designed to build powerful, composable AI-powered applications with ruthless simplicity. It provides the building blocks for creating sophisticated AI agents while maintaining clean architecture and developer ergonomics.

## Overview

At its core, Amplifier is a **kernel-based architecture** that orchestrates AI agents through a minimal, well-defined set of contracts. Rather than providing a monolithic framework, Amplifier offers a thin kernel surrounded by composable modules that can be mixed and matched to create exactly the capabilities you need.

Think of it like building with construction bricks: the kernel provides the studs and sockets (the connection points), while modules snap together to form complete applications. Each piece is self-contained, testable, and replaceable without affecting the rest of the system.

```
┌─────────────────────────────────────────────────┐
│                  Your Application               │
├─────────────────────────────────────────────────┤
│     Bundles (Pre-composed Module Sets)          │
├─────────────────────────────────────────────────┤
│  Providers  │  Tools  │  Hooks  │  Behaviors    │
├─────────────────────────────────────────────────┤
│              Amplifier Kernel                   │
└─────────────────────────────────────────────────┘
```

## Key Features

### Modular Architecture

Amplifier's module system allows you to compose functionality from discrete, well-defined pieces:

- **Providers**: Connect to AI models (Anthropic, OpenAI, Azure, local models)
- **Tools**: Extend agent capabilities (file operations, web search, code execution)
- **Hooks**: Intercept and modify agent behavior at key lifecycle points
- **Behaviors**: Reusable patterns for common agent tasks

### Thin Bundles

Bundles are pre-composed collections of modules that work well together. They follow the "thin bundle" pattern: minimal configuration that composes existing modules rather than implementing new functionality. This keeps complexity low while enabling powerful combinations.

### Session Management

Amplifier provides robust session handling for multi-turn conversations:

- Automatic context persistence
- Resumable sessions across restarts
- Event-based history with JSONL storage
- Sub-session support for agent delegation

### Multi-Agent Patterns

Build sophisticated multi-agent systems with built-in support for:

- Agent delegation (spawning sub-agents for specialized tasks)
- Parallel agent execution
- Hierarchical agent orchestration
- Shared context and memory

### Tool Ecosystem

A rich ecosystem of tools ready to use:

- File system operations (read, write, edit, search)
- Web capabilities (search, fetch, scrape)
- Code intelligence (LSP integration, semantic navigation)
- Git and GitHub operations
- Shell command execution with safety guardrails

### Recipes

Declarative YAML workflows for multi-step agent tasks:

- Sequential execution with state persistence
- Approval gates for human-in-the-loop workflows
- Automatic checkpointing and resumability
- Error handling and retry logic

## Philosophy

Amplifier is built on a foundation of intentional simplicity and practical wisdom.

### Ruthless Simplicity

Every line of code must justify its existence. Amplifier rejects complexity for its own sake:

- **Start minimal**: Begin with the simplest implementation that works
- **Grow as needed**: Add complexity only when requirements demand it
- **Question everything**: Regularly challenge assumptions and abstractions

### The Brick Philosophy

Software is built from small, clear modules like interlocking bricks:

- Each module is **self-contained** with defined interfaces
- Modules can be **regenerated independently** without breaking the system
- External contracts (the "studs and sockets") remain **stable**
- Internal implementations can evolve freely

### Humans as Architects

Amplifier elevates human involvement to where it's most valuable:

- **Humans define the vision**: Specifications, requirements, intended behavior
- **AI handles construction**: Code generation, implementation details
- **Humans validate results**: Testing behavior, not reading every line of code

### Present-Moment Focus

Build for what's needed now, not hypothetical futures:

- Handle current requirements well
- Avoid premature optimization
- Trust that good architecture emerges from simplicity

## Who It's For

### AI Application Developers

If you're building AI-powered applications, Amplifier provides:

- Clean abstractions for common patterns
- Provider-agnostic model integration
- Battle-tested tool implementations
- Extensible architecture for custom needs

### Framework Builders

If you're creating AI frameworks or platforms:

- Study Amplifier's kernel architecture
- Understand module contracts and composition
- Learn from the "thin bundle" pattern
- Build compatible modules and extensions

### Power Users

If you're using AI assistants and want more control:

- Configure bundles to match your workflow
- Create custom tools and behaviors
- Build specialized agents for your domain
- Integrate with your existing systems

### Teams Exploring AI

If your team is evaluating AI development approaches:

- Amplifier demonstrates sustainable AI development patterns
- The modular architecture prevents vendor lock-in
- Clear contracts make testing and validation straightforward
- Philosophy documentation guides decision-making

## Core Concepts at a Glance

| Concept | Description |
|---------|-------------|
| **Kernel** | Minimal core that orchestrates modules and sessions |
| **Module** | Self-contained unit of functionality (provider, tool, hook, behavior) |
| **Bundle** | Pre-composed collection of modules for specific use cases |
| **Session** | A conversation context with history and state |
| **Agent** | An AI entity that can use tools and follow instructions |
| **Recipe** | Declarative workflow specification in YAML |

## What Amplifier is NOT

Understanding what Amplifier isn't helps set proper expectations:

- **Not a chatbot UI**: Amplifier is a framework, not an application
- **Not a model trainer**: It orchestrates models, doesn't train them
- **Not opinionated about models**: Works with any compatible provider
- **Not a low-code platform**: Designed for developers who code
- **Not trying to solve everything**: Focused on agent orchestration

## Next Steps

Ready to get started? Here's your path forward:

1. **[Installation](./installation.md)**: Set up Amplifier in your environment
2. **[First Conversation](./first-conversation.md)**: Start your first session
3. **[Your First Bundle](./first-bundle.md)**: Create custom behavior packages
4. **[Concepts](../concepts/index.md)**: Deeper dive into architecture

### Quick Links

- [GitHub Repository](https://github.com/microsoft/amplifier) - Source code and issues
- [Tools Reference](../tools/index.md) - Complete documentation of available tools
- [Bundles](../bundles/index.md) - Pre-built capability packages

---

**Welcome to Amplifier.** Build powerful AI agents with clarity and confidence.
