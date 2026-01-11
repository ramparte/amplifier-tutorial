---
id: concepts-index
type: section-index
title: Core Concepts
---

# Core Concepts

Understanding Amplifier's architecture and design philosophy is key to building effective AI-powered applications. This section explains the foundational concepts that power the system.

These concepts build on each other—start with the basics and progress to more advanced architectural patterns.

## Section Contents

| Page | Description |
|------|-------------|
| [Sessions](./sessions.md) | Conversation state and lifecycle management |
| [Agents](./agents.md) | Autonomous task executors with tools |
| [Providers](./providers.md) | LLM backends (Anthropic, OpenAI, Azure) |
| [Modules](./modules.md) | Extensibility through the module system |
| [Hooks](./hooks.md) | Event interception and custom behaviors |
| [Context](./context.md) | How agents receive and manage information |
| [Tool Architecture](./tool-architecture.md) | How tools work under the hood |
| [Multi-Agent Patterns](./multi-agent.md) | Coordination between multiple agents |

## Quick Tips

- **Sessions are stateful** — Each session maintains full conversation history
- **Agents are stateless** — Agent definitions are templates; sessions hold state
- **Modules are composable** — Mix and match capabilities without conflicts
- **Hooks are powerful** — Intercept any event for custom behavior
- **Context is king** — What the agent knows determines what it can do

## Core Architecture

```
┌─────────────────────────────────────────┐
│              Application                │
├─────────────────────────────────────────┤
│    Bundles (composed configurations)    │
├─────────────────────────────────────────┤
│  Modules: Providers | Tools | Hooks     │
├─────────────────────────────────────────┤
│           Amplifier Kernel              │
└─────────────────────────────────────────┘
```

## Key Principles

| Principle | Description |
|-----------|-------------|
| Thin Bundles | Bundles compose modules; they don't implement |
| Module Contracts | Clear interfaces between components |
| Event-Driven | Hooks respond to lifecycle events |
| Minimal Core | Kernel is small; features live in modules |

## Where to Start

**Just getting started?** Read [Sessions](./sessions.md) first — it's the foundation for everything else and explains how conversations work.

**Want to customize behavior?** Jump to [Hooks](./hooks.md) to learn how to intercept and modify agent behavior.

**Building multi-agent systems?** Start with [Agents](./agents.md) then proceed to [Multi-Agent Patterns](./multi-agent.md).

## Concept Dependencies

```
Sessions ─────┬──► Agents ──► Multi-Agent
              │
Providers ────┤
              │
Modules ──────┼──► Hooks
              │
Context ──────┴──► Tool Architecture
```

## Related Sections

- [Quickstart: Your First Agent](../quickstart/first-agent.md)
- [Advanced: Kernel Internals](../advanced/kernel-internals.md)
- [Bundles: Bundle Composition](../bundles/composition.md)
