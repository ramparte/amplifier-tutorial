---
id: what-is-amplifier
type: quickstart
title: "What is Amplifier?"
---

# What is Amplifier?

## Overview

Amplifier is a modular AI agent framework designed to help developers build powerful,
composable AI applications. Rather than providing a monolithic solution, Amplifier
embraces a "bricks and studs" philosophy where self-contained modules snap together
to create sophisticated agent systems.

At its core, Amplifier provides:

- **A minimal kernel** that handles the essential agent loop, session management, and
  tool execution
- **A module system** with well-defined contracts for extending functionality
- **A bundle architecture** for packaging and sharing agent configurations
- **Built-in support** for multiple LLM providers, tools, and agent patterns

Amplifier is built for developers who want full control over their AI agents without
drowning in boilerplate or fighting framework constraints.


## Key Features

### Modular Architecture

Every component in Amplifier is a self-contained module with clear boundaries:

```
┌─────────────────────────────────────────────────┐
│                  Amplifier Kernel               │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐      │
│  │ Provider │  │  Tools   │  │  Hooks   │      │
│  └──────────┘  └──────────┘  └──────────┘      │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐      │
│  │ Context  │  │  Skills  │  │  Agents  │      │
│  └──────────┘  └──────────┘  └──────────┘      │
└─────────────────────────────────────────────────┘
```

Modules communicate through stable interfaces, allowing you to:

- Swap providers (Anthropic, OpenAI, Azure, local models)
- Add or remove tools without touching other code
- Create custom hooks for observability and control
- Package everything into shareable bundles

### Bundle System

Bundles are the primary unit of composition in Amplifier. A bundle packages:

- Agent definitions and configurations
- Tool collections for specific capabilities
- Context files and instructions
- Skills and domain knowledge

```yaml
# Example bundle structure
my-bundle/
├── bundle.yaml        # Bundle manifest
├── agents/            # Agent definitions
├── tools/             # Custom tools
├── context/           # Instructions and context
└── skills/            # Domain knowledge
```

Bundles can compose other bundles, enabling a layered architecture where
specialized capabilities build on foundational ones.

### Multi-Agent Support

Amplifier supports sophisticated multi-agent patterns:

- **Task delegation**: Spawn specialized agents for focused work
- **Parallel execution**: Run multiple agents concurrently
- **Agent hierarchies**: Build orchestration patterns
- **Session continuity**: Resume and branch conversations

### Rich Tool Ecosystem

Built-in tools cover common development needs:

- File operations (read, write, edit, search)
- Shell command execution
- Web search and content fetching
- Git and GitHub operations
- LSP integration for code intelligence

Custom tools are straightforward to create and package.

### Session Management

Amplifier provides robust session handling:

- Automatic checkpointing and resumability
- Event logging for debugging and analysis
- Context management across long conversations
- Sub-session spawning for agent delegation


## Philosophy

Amplifier is built on three core principles:

### 1. Ruthless Simplicity

> "The best code is often the simplest."

Amplifier resists the urge to over-engineer. Every abstraction must justify its
existence. The framework provides what's essential and stays out of your way
for everything else.

This means:

- Minimal magic and hidden behavior
- Clear, predictable execution paths
- Easy-to-understand source code
- Simple extension patterns

### 2. Composability Over Configuration

Rather than exposing hundreds of configuration options, Amplifier achieves
flexibility through composition. Small, focused modules combine to create
exactly the agent system you need.

Think of it like building blocks:

- Each block does one thing well
- Blocks connect through standard interfaces
- Complex systems emerge from simple parts
- Any block can be replaced or regenerated

### 3. AI-Native Development

Amplifier embraces the reality that AI is changing how we build software.
The framework is designed to work with AI assistants:

- Modules are sized for AI comprehension
- Specifications drive implementation
- Regeneration is preferred over patching
- Clear contracts enable reliable generation


## Who It's For

### AI Application Developers

If you're building AI-powered applications and want:

- Full control over agent behavior
- Easy integration with multiple providers
- Extensible tool and capability systems
- Clean separation of concerns

Amplifier provides the foundation without imposing unnecessary constraints.

### Teams Building Internal Tools

For organizations creating internal AI assistants:

- Package domain knowledge into bundles
- Define organization-specific tools
- Control security and access patterns
- Share capabilities across teams

### Researchers and Experimenters

If you're exploring agent architectures:

- Easily swap components to test hypotheses
- Clear extension points for custom behavior
- Observable execution for analysis
- Minimal overhead for rapid iteration

### Open Source Contributors

Amplifier's modular design makes contribution straightforward:

- Work on isolated modules without deep framework knowledge
- Clear contracts define expected behavior
- Test modules independently
- Share bundles with the community


## Core Concepts

Before diving deeper, familiarize yourself with these key terms:

| Concept | Description |
|---------|-------------|
| **Kernel** | The minimal core that runs the agent loop |
| **Module** | A self-contained unit of functionality |
| **Bundle** | A package of agents, tools, and configuration |
| **Provider** | An LLM backend (Anthropic, OpenAI, etc.) |
| **Tool** | A capability the agent can invoke |
| **Hook** | An extension point for observing or modifying behavior |
| **Skill** | Loadable domain knowledge and best practices |
| **Session** | A conversation instance with state and history |
| **Agent** | A configured AI assistant with specific capabilities |


## Next Steps

Ready to get started? Here's your path forward:

1. **[Installation](./installation.md)** - Set up Amplifier in your environment

2. **[Your First Agent](./first-agent.md)** - Build and run a simple agent

3. **[Understanding Bundles](./bundles-intro.md)** - Learn the composition model

4. **[Core Concepts Deep Dive](../concepts/overview.md)** - Explore the architecture

5. **[Building Tools](../guides/custom-tools.md)** - Extend agent capabilities

---

Amplifier is open source and actively developed. Join the community to ask
questions, share bundles, and contribute to the framework's evolution.
