---
id: concepts-index
type: section-index
title: Core Concepts
---

# Core Concepts

Understanding Amplifier's architecture and design philosophy is essential for building effective AI-powered applications. These concepts form the mental model you'll use throughout your work. This section explains the foundational ideas, patterns, and principles that make Amplifier work.

## Section Contents

| Page | Description |
|------|-------------|
| [Agents](./agents.md) | Autonomous task executors with specific capabilities |
| [Bundles](./bundles.md) | Composable packages of behaviors and tools |
| [Modules](./modules.md) | Kernel extension points and contracts |
| [Hooks](./hooks.md) | Event-driven extensibility system |
| [Skills](./skills.md) | Domain knowledge packages |
| [Recipes](./recipes.md) | Declarative multi-step workflows |
| [Architecture](./architecture.md) | System design and philosophy |

## Quick Tips

- **Sessions are stateful** - Each conversation maintains history and can be resumed
- **Agents are specialists** - Delegate complex tasks to purpose-built agents
- **Bundles compose** - Layer multiple bundles to build custom capabilities
- **Modules extend** - Add new functionality without modifying the kernel
- **Context is king** - What the model sees determines what it can do

## Key Principles

### Modular Design
Everything in Amplifier is designed to be composed, replaced, and extended. No monoliths.

### Thin Bundles
Bundles should be minimal—just enough to define behavior, composed from reusable parts.

### Ruthless Simplicity
Complexity is the enemy. Every abstraction must justify its existence.

### Trust in Emergence
Complex behaviors emerge from simple, well-defined components working together.

## Where to Start

**Understanding architecture?** Read [Architecture](./architecture.md) first—understanding the system design is fundamental.

**Building custom behaviors?** Start with [Bundles](./bundles.md) to see how capabilities compose.

**Extending the system?** [Modules](./modules.md) explains the kernel's extension contracts.

## Concept Map

```
┌─────────────────────────────────────────┐
│              Application                │
├─────────────────────────────────────────┤
│  Bundle  │  Bundle  │  Bundle           │
├──────────┴──────────┴──────────────────┤
│  Agents  │  Tools   │  Context  │ Hooks │
├─────────────────────────────────────────┤
│              Kernel (Modules)           │
├─────────────────────────────────────────┤
│              Providers (LLMs)           │
└─────────────────────────────────────────┘
```

## Next Steps

Once you grasp these concepts, move to [Bundles](../bundles/index.md) to see them applied in practice.
