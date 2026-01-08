---
id: bundles-overview
type: bundles
title: "Bundles Reference"
---

# Bundles Reference

Bundles are composable packages that configure Amplifier for specific use cases.

## What Are Bundles?

A bundle packages:

- **Modules** - Tools, providers, hooks to load
- **Agents** - Specialized AI configurations
- **Behaviors** - Reusable capability add-ons
- **Instructions** - System prompts and context

## Official Bundles

| Bundle | Description | Guide |
|--------|-------------|-------|
| [Foundation](foundation.md) | Core tools, agents, philosophy | Base for all configs |
| [Recipes](recipes.md) | Multi-step workflow orchestration | Repeatable workflows |
| [LSP Python](lsp-python.md) | Python code intelligence | Semantic navigation |
| [Design Intelligence](design-intelligence.md) | 7 design specialist agents | UI/UX design work |

## Quick Comparison

| Bundle | Tools Added | Agents Added | Use Case |
|--------|-------------|--------------|----------|
| Foundation | Core 10+ | 12 specialists | Everything |
| Recipes | Recipe executor | Recipe author | Workflows |
| LSP Python | LSP operations | Python code intel | Python projects |
| Design Intelligence | - | 7 design agents | Design work |

## Managing Bundles

### Install a Bundle

```bash
amplifier bundle add git+https://github.com/microsoft/amplifier-bundle-recipes@main
```

### Activate a Bundle

```bash
amplifier bundle use recipes
```

### List Installed Bundles

```bash
amplifier bundle list
```

### Remove a Bundle

```bash
amplifier bundle remove recipes
```

## Bundle Composition

Bundles can include other bundles:

```yaml
# my-bundle/bundle.yaml
bundle:
  name: my-custom
  version: 1.0.0

includes:
  - bundle: foundation
  - bundle: recipes
  - bundle: lsp-python

# Add your own on top
agents:
  - path: ./agents/my-agent.yaml
```

## Creating Custom Bundles

See [Advanced: Custom Bundles](../advanced/custom-bundle.md) for:

- Bundle structure
- Including other bundles
- Adding tools and agents
- Configuration options

## Bundle Sources

```yaml
includes:
  # Named bundle (from registry/installed)
  - bundle: foundation
  
  # From GitHub
  - bundle: git+https://github.com/org/bundle@main
  
  # From local path
  - bundle: ./my-local-bundle
```

## Community Bundles

Community-contributed bundles:

| Bundle | Author | Description |
|--------|--------|-------------|
| `lsp-typescript` | robotdad | TypeScript code intelligence |
| `dev-memory` | community | Persistent memory system |

To use community bundles:

```bash
amplifier bundle add git+https://github.com/[user]/[bundle]@main
```
