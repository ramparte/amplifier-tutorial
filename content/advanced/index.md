---
id: advanced-index
type: section-index
title: Advanced Topics
---

# Advanced Topics

This section covers advanced Amplifier concepts for users who want to go deeper. Topics here assume familiarity with core concepts, bundles, and basic agent development.

These guides help you build sophisticated AI applications, customize the kernel, and optimize performance.

## Section Contents

| Page | Description |
|------|-------------|
| [Kernel Internals](./kernel-internals.md) | How the Amplifier kernel works |
| [Custom Tools](./custom-tools.md) | Building your own tool modules |
| [Hook Development](./hook-development.md) | Creating custom hooks |
| [Provider Integration](./provider-integration.md) | Adding new LLM providers |
| [Multi-Agent Systems](./multi-agent-systems.md) | Complex agent orchestration |
| [Recipes](./recipes.md) | Declarative multi-step workflows |
| [Performance](./performance.md) | Optimization and scaling |
| [Security](./security.md) | Security model and best practices |

## Quick Tips

- **Understand modules first** — All advanced features build on the module system
- **Read the source** — Amplifier is open; the code is the best documentation
- **Test in isolation** — Advanced features can have subtle interactions
- **Use typed interfaces** — TypedDict and Protocol classes catch errors early
- **Profile before optimizing** — Measure actual bottlenecks, don't guess

## Architecture Deep Dive

```
┌─────────────────────────────────────────────────────────┐
│                    Application Layer                    │
├─────────────────────────────────────────────────────────┤
│  Recipes │ Multi-Agent │ Custom Behaviors               │
├─────────────────────────────────────────────────────────┤
│      Modules: Providers │ Tools │ Hooks │ Custom        │
├─────────────────────────────────────────────────────────┤
│                   Event System                          │
├─────────────────────────────────────────────────────────┤
│    Kernel: Session Manager │ Module Registry │ Router   │
└─────────────────────────────────────────────────────────┘
```

## Module Types

| Type | Purpose | Interface |
|------|---------|-----------|
| Provider | LLM backends | `ProviderProtocol` |
| Tool | Agent capabilities | `ToolProtocol` |
| Hook | Event interception | `HookProtocol` |
| Context | Knowledge injection | `ContextProtocol` |

## Where to Start

**Want to extend Amplifier?** Start with [Custom Tools](./custom-tools.md)—it's the most common extension point.

**Building complex workflows?** Read [Recipes](./recipes.md) for declarative multi-step orchestration.

**Optimizing performance?** Check [Performance](./performance.md) for profiling and optimization strategies.

**Security concerns?** Review [Security](./security.md) before deploying to production.

## Advanced Patterns

| Pattern | Use Case | Guide |
|---------|----------|-------|
| Tool composition | Combine tools for complex ops | Custom Tools |
| Event sourcing | Audit and replay | Kernel Internals |
| Agent delegation | Hierarchical task distribution | Multi-Agent Systems |
| Staged execution | Human-in-loop workflows | Recipes |

## Prerequisites

Before diving into advanced topics, ensure you understand:

- [ ] Core concepts (sessions, agents, modules)
- [ ] Bundle composition and structure
- [ ] Basic tool usage patterns
- [ ] Hook event lifecycle

## Related Sections

- [Concepts: Core Architecture](../concepts/index.md)
- [Dev Setup: Contributing](../dev-setup/contributing.md)
- [Bundles: Creating Bundles](../bundles/creating.md)
