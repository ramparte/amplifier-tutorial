---
id: bundles-index
type: section-index
title: Bundles Guide
---

# Bundles Guide

Bundles are the primary way to configure and customize Amplifier. A bundle composes modules, tools, context, and settings into a coherent package that defines an agent's capabilities.

This section covers bundle structure, composition patterns, and how to create your own bundles.

## Section Contents

| Page | Description |
|------|-------------|
| [Bundle Basics](./basics.md) | What bundles are and how they work |
| [Built-in Bundles](./built-in.md) | Foundation, dev, and other standard bundles |
| [Bundle Structure](./structure.md) | Anatomy of a bundle directory |
| [Composition](./composition.md) | Combining multiple bundles |
| [Creating Bundles](./creating.md) | Build your own custom bundles |
| [Context Files](./context-files.md) | Adding knowledge to bundles |
| [Tool Bundles](./tool-bundles.md) | Bundles that provide tools |
| [Publishing](./publishing.md) | Sharing bundles with others |

## Quick Tips

- **Thin bundles** — Bundles should compose, not implement; keep logic in modules
- **Layered composition** — Bundles can extend other bundles for customization
- **Context is cheap** — Add relevant context files liberally; they help agents
- **One purpose** — Each bundle should have a clear, focused purpose
- **Test composition** — Verify bundles work together before deploying

## Bundle Architecture

```
bundle/
├── bundle.yaml          # Bundle manifest
├── context/             # Context files (.md)
│   ├── instructions.md
│   └── domain-knowledge.md
├── skills/              # Packaged skills
├── agents/              # Agent definitions
└── tools/               # Custom tools (optional)
```

## Built-in Bundles

| Bundle | Purpose | Key Features |
|--------|---------|--------------|
| `foundation` | General development | Full toolset, explorer, git-ops |
| `dev` | Lightweight coding | Essential tools, minimal context |
| `core` | Minimal base | Kernel only, for custom builds |
| `lsp-python` | Python intelligence | LSP integration, Pyright |

## Where to Start

**New to bundles?** Begin with [Bundle Basics](./basics.md) to understand what bundles are and why they matter.

**Want to customize?** Read [Creating Bundles](./creating.md) for a step-by-step guide to building your own.

**Using multiple bundles?** Check [Composition](./composition.md) to learn how bundles layer and interact.

## Common Patterns

```yaml
# Extend an existing bundle
extends: foundation

# Add custom context
context:
  - ./context/my-domain.md
  
# Include specific modules
modules:
  - tool-web
  - hook-memory
```

## Bundle Selection Guide

| Use Case | Recommended Bundle |
|----------|-------------------|
| General development | `foundation` |
| Quick coding tasks | `dev` |
| Python projects | `foundation` + `lsp-python` |
| Custom applications | Create your own |
| Minimal footprint | `core` + specific modules |

## Related Sections

- [Concepts: Modules](../concepts/modules.md)
- [Skills: Using Skills](../skills/index.md)
- [Advanced: Bundle Internals](../advanced/bundle-internals.md)
