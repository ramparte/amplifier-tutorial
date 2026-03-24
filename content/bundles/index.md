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
| [Foundation](./foundation.md) | Core Amplifier bundle with essential capabilities |
| [Recipes](./recipes.md) | Multi-step workflow orchestration |
| [LSP Python](./lsp-python.md) | Python code intelligence via Language Server Protocol |
| [Design Intelligence](./design-intelligence.md) | Design system and UI expertise |

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

**Core capabilities?** Begin with [Foundation](./foundation.md) for the essential Amplifier bundle.

**Workflow automation?** Jump to [Recipes](./recipes.md) for multi-step orchestration.

**Python development?** See [LSP Python](./lsp-python.md) for code intelligence.

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
