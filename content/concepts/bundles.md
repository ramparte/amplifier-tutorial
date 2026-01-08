---
id: bundles
type: concepts
title: "Bundles"
---

# Bundles

Bundles are the primary way to configure and extend Amplifier. They package tools, agents, behaviors, and instructions into composable units.

## What is a Bundle?

A bundle is a YAML configuration that tells Amplifier:

- Which **modules** to load (tools, providers, hooks)
- Which **agents** are available
- What **instructions** guide the AI's behavior
- What **behaviors** add specialized capabilities

Think of bundles like Docker Compose files - they define a complete environment from reusable pieces.

## Bundle Structure

```yaml
# bundle.yaml
bundle:
  name: my-bundle
  version: 1.0.0
  description: My custom Amplifier configuration

# Include other bundles
includes:
  - bundle: foundation  # Base bundle with core tools

# Add tools
tools:
  - module: tool-filesystem
  - module: tool-bash
  - module: my-custom-tool
    source: ./modules/my-tool

# Add agents
agents:
  - path: ./agents/specialist.yaml

# Add behaviors
behaviors:
  - path: ./behaviors/code-review.yaml

# System instructions
instructions: |
  You are a helpful assistant specialized in Python development.
  Always suggest type hints and write tests.
```

## The Foundation Bundle

Most bundles start by including `foundation`:

```yaml
includes:
  - bundle: foundation
```

Foundation provides:

| Component | What You Get |
|-----------|--------------|
| **Core Tools** | filesystem, bash, web, search, task, todo |
| **Agents** | zen-architect, bug-hunter, modular-builder, explorer, etc. |
| **Behaviors** | Git operations, security review, test coverage |
| **Philosophy** | Implementation and modular design principles |

## Using Bundles

### Install a Bundle

```bash
# Add a bundle from GitHub
amplifier bundle add git+https://github.com/microsoft/amplifier-bundle-recipes@main

# List installed bundles
amplifier bundle list
```

### Activate a Bundle

```bash
# Use a specific bundle for this session
amplifier bundle use recipes

# Or specify inline
amplifier --bundle recipes
```

### View Bundle Contents

```bash
# See what a bundle provides
amplifier bundle show recipes
```

## Bundle Composition

Bundles can include other bundles, creating a composition hierarchy:

```
your-bundle
├── includes: recipes
│   └── includes: foundation
│       └── (core tools, agents, behaviors)
├── your tools
├── your agents
└── your instructions
```

### Example: Team Bundle

```yaml
# team-bundle/bundle.yaml
bundle:
  name: acme-dev
  version: 1.0.0

includes:
  - bundle: foundation
  - bundle: recipes
  - bundle: lsp-python

# Team-specific tools
tools:
  - module: tool-jira
    source: ./modules/jira-integration

# Team-specific agents
agents:
  - path: ./agents/code-reviewer.yaml
  - path: ./agents/deploy-manager.yaml

# Team coding standards
instructions: |
  Follow ACME coding standards:
  - Use Google-style docstrings
  - All functions must have type hints
  - Tests required for new code
```

## Bundle Sources

Bundles can be loaded from multiple sources:

```yaml
includes:
  # From the registry (if configured)
  - bundle: foundation
  
  # From GitHub
  - bundle: git+https://github.com/microsoft/amplifier-bundle-recipes@main
  
  # From local path
  - bundle: ./my-local-bundle
  
  # From specific version/tag
  - bundle: git+https://github.com/org/bundle@v1.2.0
```

## Bundle vs Module vs Behavior

| Concept | Purpose | Scope |
|---------|---------|-------|
| **Module** | Single capability (tool, provider, hook) | One thing |
| **Behavior** | Reusable agent + context combo | One workflow |
| **Bundle** | Complete configuration package | Everything |

A bundle can contain multiple behaviors, which reference multiple modules.

## Try It Yourself

### Exercise 1: Explore Your Current Bundle

```bash
# Start Amplifier and ask:
amplifier

> What bundle am I using? What tools and agents are available?
```

### Exercise 2: Add the Recipes Bundle

```bash
# Add and use the recipes bundle
amplifier bundle add git+https://github.com/microsoft/amplifier-bundle-recipes@main
amplifier bundle use recipes

# Verify it's loaded
amplifier run "/tools" | grep recipe
```

### Exercise 3: Create a Minimal Bundle

Create a file `my-bundle/bundle.yaml`:

```yaml
bundle:
  name: minimal
  version: 1.0.0

includes:
  - bundle: foundation

instructions: |
  You are a concise assistant.
  Always give brief answers unless asked for detail.
```

Use it:
```bash
amplifier --bundle ./my-bundle
```

## Key Takeaways

1. **Bundles are configuration** - They assemble modules, agents, and instructions
2. **Composition is powerful** - Include other bundles to build on their capabilities
3. **Foundation is your base** - Most bundles include it for core functionality
4. **Bundles are portable** - Share via Git, zip, or any file transfer

## Next

Learn about the building blocks that bundles assemble:

→ [Modules](modules.md)
