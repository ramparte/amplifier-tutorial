---
id: what-is-amplifier
type: quickstart
title: "What is Amplifier?"
---

# What is Amplifier?

Amplifier is a **modular AI agent framework** that lets you build, customize, and control AI-powered development tools.

Think of it as the difference between renting a car (most AI tools) and owning one you can modify (Amplifier). You get full control over the engine, the interior, and every component.

## The Linux Kernel Philosophy

Amplifier is built on a principle borrowed from Linux: **a tiny, stable kernel with swappable modules**.

```
┌─────────────────────────────────────────┐
│           Your Application              │
├─────────────────────────────────────────┤
│  Bundles (recipes, lsp-python, etc.)    │
├─────────────────────────────────────────┤
│  Modules (tools, providers, hooks)      │
├─────────────────────────────────────────┤
│         Amplifier Kernel                │
│       (~2,600 lines of code)            │
└─────────────────────────────────────────┘
```

The kernel is intentionally minimal. It handles:

- Loading and unloading modules
- Managing sessions and context
- Emitting events for observability
- Coordinating between components

Everything else - which AI model to use, what tools are available, how to orchestrate complex workflows - lives in **modules** that you can swap, customize, or build yourself.

## Key Concepts (Preview)

### Bundles
Composable packages that configure Amplifier for specific use cases. Want Python development tools? Add the `lsp-python` bundle. Need multi-step workflows? Add the `recipes` bundle.

### Modules
The building blocks: providers (AI backends), tools (capabilities), hooks (observers), and orchestrators (execution strategies).

### Agents
Specialized AI configurations for specific tasks. The `bug-hunter` agent debugs issues. The `zen-architect` agent designs solutions. You can create your own.

### Recipes
Declarative YAML workflows that orchestrate multiple agents across multiple steps. They can pause for approval, resume after interruption, and accumulate context.

## What Can You Build?

- **Custom coding assistants** tailored to your stack
- **Automated workflows** (code review, dependency updates, security audits)
- **Multi-agent systems** where specialists collaborate
- **Development environments** with your preferred tools and providers

## Try It Now

```bash
# See what Amplifier can do
amplifier run "What tools do you have available?"
```

You'll see a response listing all the capabilities currently loaded - file operations, shell commands, web search, and more. These are all modules you can configure, extend, or replace.

## Next

Ready to understand how this differs from tools you might already use?

→ [How It's Different](how-its-different.md)
