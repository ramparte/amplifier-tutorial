---
id: foundation
type: bundle
title: "Foundation Bundle"
---

# Foundation Bundle

Every Amplifier bundle you'll ever build starts the same way: it includes Foundation. This is the base layer of the entire ecosystem -- the bundle that provides file operations, shell access, search, web research, streaming UI, and over twenty specialized agents. When you type a prompt and Amplifier reads a file, runs a test, or delegates to a sub-agent, that's Foundation at work.

## What is Foundation?

Foundation is the primary library for building Amplifier applications. It's both a Python package (bundle loading, composition, validation, utilities) and a reference bundle (agents, behaviors, context, provider configs). Most bundles in the ecosystem are *thin* -- they include Foundation and add only what's unique to their purpose.

The design philosophy is **mechanism, not policy**. Foundation provides the loading and composition machinery. It provides agents and behaviors. But it doesn't decide how you use them -- your bundle does.

## What's Included

Foundation ships four categories of content:

| Category | What It Provides |
|----------|-----------------|
| **Bundle System** | `load_bundle()`, `Bundle.compose()`, validation, `@mention` resolution |
| **Reference Content** | 20+ agents, 4 behavior groups, provider configs, shared context |
| **Utilities** | YAML/frontmatter I/O, deep merge, path handling, caching |
| **Examples** | 22+ examples from hello-world to full workflow orchestration |

The bundle itself declares tools (filesystem, bash, search, web, task delegation), hooks (streaming UI, status reporting), the orchestrator, and all the shared context that makes Amplifier sessions work out of the box.

## Getting Started

Foundation is loaded automatically when you include it in a bundle. The most common pattern is a thin bundle that inherits everything:

```markdown
---
bundle:
  name: my-assistant
  version: 1.0.0

includes:
  - bundle: git+https://github.com/microsoft/amplifier-foundation@main
---

# My Assistant

Custom instructions go here.

---

@foundation:context/shared/common-system-base.md
```

That's a complete, working bundle. All of Foundation's tools, agents, hooks, and streaming UI are inherited. Your bundle adds only its own instructions.

> Show me what tools are available

```
[Tool: bash] amplifier tools list
  bash, read_file, write_file, edit_file, grep, glob, web_search,
  web_fetch, python_check, LSP, recipes, todo, load_skill, mode, ...
```

Everything listed comes from Foundation. You didn't declare any of it -- composition handled that.

## Key Agents

Foundation includes over twenty agents, each a specialist that carries its own context and instructions. Here are the ones you'll use most:

| Agent | Role |
|-------|------|
| **explorer** | Codebase reconnaissance -- maps structure, dependencies, and entry points |
| **zen-architect** | System design and architecture planning |
| **modular-builder** | Feature implementation following best practices |
| **bug-hunter** | Debugging, root cause analysis, and fix implementation |
| **git-ops** | Branch management, commits, and PR workflows |
| **security-guardian** | Vulnerability scanning and security review |
| **web-research** | Internet research for docs, examples, and solutions |
| **file-ops** | Bulk file operations -- rename, move, restructure |
| **test-coverage** | Test strategy, coverage analysis, and test writing |
| **integration-specialist** | Connecting services, APIs, and third-party tools |
| **post-task-cleanup** | Code cleanup -- imports, formatting, dead code removal |
| **session-analyst** | Conversation history analysis and insight extraction |
| **ecosystem-expert** | Technology guidance for APIs, frameworks, and libraries |
| **foundation-expert** | Guidance on building bundles with Foundation itself |
| **amplifier-smoke-test** | End-to-end verification of Amplifier installations |

These agents are invoked through the task tool. You describe what you need, and Amplifier delegates:

> Investigate why the login tests are failing

```
[Tool: task] Delegating to bug-hunter...
  Analyzing test_auth.py::test_login
  Found: mock for token service returns expired timestamp
  Fix applied in auth/token_service.py line 47
  All 12 auth tests now pass.
```

> Design a caching layer for the API

```
[Tool: task] Delegating to zen-architect...
  Analyzed current request flow (3 database calls per request)
  Proposed: two-tier cache (in-memory LRU + Redis)
  Created design spec with interface contracts and eviction strategy.
```

Agents chain naturally. A common workflow: **explorer** maps the territory, **zen-architect** designs the approach, **modular-builder** implements it, **test-coverage** validates it, and **git-ops** ships it.

## Key Behaviors

Behaviors are reusable capability packages -- bundles of agents, context, and tools grouped by domain. Foundation organizes its reference content into four behavior groups:

**agents** -- The delegation patterns. This behavior includes all of Foundation's agents and the context that teaches Amplifier *when* to delegate versus handle directly. It's why Amplifier knows to call bug-hunter for a failing test and zen-architect for a design question.

**filesystem** -- File reading, writing, editing, and the `@mention` resolution system for cross-bundle file references.

**web** -- Web search and fetch capabilities, plus the instructions that guide responsible web usage.

**search** -- The grep and glob tools with their configuration for ignoring build artifacts, respecting `.gitignore`, and paginating large result sets.

You can include individual behaviors rather than the entire bundle when composing lightweight configurations:

```yaml
includes:
  - bundle: foundation:behaviors/agents
  - bundle: foundation:behaviors/filesystem
```

## Composition Patterns

Foundation's architecture enables three key patterns for building bundles:

### Thin Bundles

The most important pattern. Your bundle declares `includes:` and custom instructions -- nothing else. Foundation provides all the machinery. This is how most production bundles work:

```yaml
includes:
  - bundle: git+https://github.com/microsoft/amplifier-foundation@main
  - bundle: my-bundle:behaviors/my-capability
```

Two lines of includes. Everything else is inherited.

### Context Sinks

Agents carry heavy documentation so your root session stays lean. When Foundation's `zen-architect` is invoked, it loads its own architecture context, design principles, and decision frameworks. That context lives inside the agent's session -- not yours. Your root session prompt stays small and fast.

This is why Foundation can have 20+ agents without bloating every conversation. Each agent is a **context sink**: it absorbs domain-specific knowledge only when activated.

> Plan a migration from REST to GraphQL

```
[Tool: task] Delegating to zen-architect...
  (zen-architect loads architecture context, API design patterns,
   migration strategies -- none of this enters your root session)
```

### Behavior Composition

Build capabilities by mixing behaviors from multiple bundles:

```yaml
includes:
  - bundle: git+https://github.com/microsoft/amplifier-foundation@main
  - bundle: git+https://github.com/microsoft/amplifier-bundle-recipes@main
  - bundle: my-bundle:behaviors/data-pipeline
```

Foundation provides the base. Recipes adds workflow orchestration. Your behavior adds domain agents. The result is a focused assistant that inherits everything and adds only what's unique.

## The Examples Directory

Foundation's repository includes 22+ examples that progress from basics to advanced patterns:

| Range | What You'll Learn |
|-------|-------------------|
| `01_hello_world` | Minimal working session |
| `04_load_and_inspect` | Loading bundles from git URLs, local paths, and registries |
| `05_composition` | Merging bundles and understanding override rules |
| `06_sources_and_registry` | Git URLs, BundleRegistry, and source management |
| `07_full_workflow` | Complete pipeline: prepare, create session, execute |
| Advanced examples | Custom providers, behavior authoring, agent creation |

These are Python scripts you can run directly. They're the fastest way to understand how Foundation's API works in practice.

## Tips

- **Start with Foundation, add sparingly.** Most bundles need nothing beyond `includes: foundation` plus custom instructions. Resist the urge to redeclare tools Foundation already provides.

- **Use agents, don't micromanage.** Instead of manually reading files, grepping, and editing, describe your goal and let the right agent handle the workflow. "Fix the failing auth tests" is better than step-by-step instructions.

- **Lean on context sinks.** If you're building a custom agent, give it rich context in its own definition file. That context loads only when the agent is invoked, keeping root sessions fast.

- **Compose behaviors, not whole bundles.** When you need only Foundation's agents (not its full tool suite), include `foundation:behaviors/agents` directly. Granular composition keeps bundles minimal.

- **Check the examples first.** Before building something from scratch, browse Foundation's `examples/` directory. There's likely a pattern that matches your use case.

## Next Steps

- Learn about [Bundles](../concepts/bundles.md) as a concept -- composition, namespaces, and merge rules
- Explore the [Recipes Bundle](./recipes.md) for multi-step workflow orchestration
- See [Design Intelligence Bundle](./design-intelligence.md) for UI/UX specialized agents
- Read about the [Task Tool](../tools/task.md) to understand how agents are delegated to
