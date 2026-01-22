---
id: bundles-index
type: section-index
title: Bundles Guide
---

# Bundles Guide

Bundles are the primary way to customize Amplifier's behavior. A bundle packages together tools, agents, context, and configuration into a cohesive, reusable unit. This section teaches you how to use existing bundles, create your own, and compose them for powerful applications.

## Section Contents

| Page | Description |
|------|-------------|
| [Bundle Basics](./bundle-basics.md) | What bundles are and how they work |
| [Using Bundles](./using-bundles.md) | Activate and configure existing bundles |
| [Creating Bundles](./creating-bundles.md) | Build your own custom bundles |
| [Bundle Composition](./bundle-composition.md) | Combine bundles effectively |
| [Context Files](./context-files.md) | Add knowledge and instructions |
| [Agent Definitions](./agent-definitions.md) | Define specialist agents in bundles |
| [Bundle Registry](./bundle-registry.md) | Discover and share bundles |
| [Best Practices](./best-practices.md) | Patterns for maintainable bundles |

## Quick Tips

- **Thin bundles** - Keep bundles focused; compose for complexity
- **Reuse context** - Reference shared context files instead of duplicating
- **Version carefully** - Bundle changes affect all users
- **Test in isolation** - Verify bundles work independently before composing
- **Document behavior** - Clear descriptions help users and AI understand intent

## Bundle Anatomy

```
my-bundle/
├── bundle.yaml        # Bundle manifest
├── context/           # Knowledge and instructions
│   ├── README.md      # Primary context
│   └── examples/      # Example files
├── agents/            # Agent definitions
│   └── specialist.yaml
└── skills/            # Optional skills
    └── domain-skill.md
```

## The Thin Bundle Philosophy

Bundles should be minimal compositions, not monolithic packages:

| Do | Don't |
|----|-------|
| Reference shared context | Duplicate instructions |
| Compose multiple thin bundles | Build one giant bundle |
| Single responsibility | Kitchen sink approach |
| Clear extension points | Tightly coupled internals |

## Where to Start

**New to bundles?** Begin with [Bundle Basics](./bundle-basics.md) for foundational understanding.

**Want to customize?** Jump to [Creating Bundles](./creating-bundles.md) for hands-on guidance.

**Combining capabilities?** See [Bundle Composition](./bundle-composition.md) for composition patterns.

## Example: Minimal Bundle

```yaml
# bundle.yaml
name: my-assistant
description: Custom assistant behavior
version: 1.0.0

context:
  - context/README.md

extends:
  - foundation  # Inherit base capabilities
```

## Next Steps

After mastering bundles, explore [Skills](../skills/index.md) for adding domain knowledge or [Advanced](../advanced/index.md) for complex patterns.
