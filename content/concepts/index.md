---
id: concepts-overview
type: concepts
title: "Core Concepts"
---

# Core Concepts

Understanding these concepts will help you get the most out of Amplifier.

## The Big Picture

Amplifier follows the **Linux kernel philosophy**: a tiny, stable core with swappable modules at the edges.

```
┌────────────────────────────────────────────────────┐
│                   Your Session                      │
├────────────────────────────────────────────────────┤
│  Bundles    │  What you've configured              │
├─────────────┴──────────────────────────────────────┤
│  Modules    │  Tools, Providers, Hooks, etc.       │
├─────────────┴──────────────────────────────────────┤
│  Kernel     │  Session lifecycle, events, loading  │
└────────────────────────────────────────────────────┘
```

## Concepts Covered

| Concept | What It Is | Why It Matters |
|---------|------------|----------------|
| [Bundles](bundles.md) | Composable configuration packages | Mix and match capabilities |
| [Modules](modules.md) | Building blocks (tools, providers, hooks) | Understand extensibility |
| [Agents](agents.md) | Specialized AI configurations | Delegate to experts |
| [Recipes](recipes.md) | Declarative multi-step workflows | Automate complex tasks |
| [Skills](skills.md) | Reusable knowledge packages | Share expertise |
| [Hooks](hooks.md) | Lifecycle observers | Control and observe |

## How They Fit Together

1. **You configure a Bundle** → Which loads Modules
2. **Modules provide capabilities** → Tools, providers, hooks
3. **Agents specialize** → Using specific tools and context
4. **Recipes orchestrate** → Multiple agents across steps
5. **Skills inform** → Agents with domain knowledge
6. **Hooks observe** → Everything that happens

## Recommended Reading Order

If you're new, read in this order:

1. **Bundles** - Start here to understand configuration
2. **Modules** - Learn the building blocks
3. **Agents** - Understand delegation
4. **Recipes** - Master workflows
5. **Skills** and **Hooks** - Advanced topics

Each page includes practical examples you can try immediately.
