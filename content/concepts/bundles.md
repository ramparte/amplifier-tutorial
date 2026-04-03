---
id: bundles
type: concept
title: "Bundles"
---

# Bundles

Every time you start an Amplifier session, something decides which tools you get, which
LLM provider handles your requests, and what instructions shape the agent's behavior.
That something is a **bundle**.

## What is a Bundle?

A bundle is a composable configuration package — the primary unit of customization in
Amplifier. It's a Markdown file with YAML frontmatter that declares everything an
AI session needs to run: tools, providers, agents, hooks, context, and instructions.

Here's the key insight: **bundles are configuration, not code.** A bundle repository
doesn't need a `pyproject.toml` at the root. It's a Markdown file that tells Amplifier
what to assemble.

A minimal bundle looks like this:

```markdown
---
bundle:
  name: my-bundle
  version: 1.0.0
  description: What this bundle provides

includes:
  - bundle: git+https://github.com/microsoft/amplifier-foundation@main
---

# My Bundle

Instructions for the AI go here as plain Markdown.
```

The YAML frontmatter declares structure. The Markdown body becomes the system prompt.

### What a Bundle Contains

A bundle can declare any combination of these:

| Component | What It Does |
|-----------|-------------|
| **includes** | Other bundles to inherit from (composition) |
| **tools** | Capabilities the agent can invoke |
| **providers** | LLM backend configurations |
| **agents** | Sub-agent definitions for task delegation |
| **hooks** | Lifecycle observers (logging, approval gates, streaming UI) |
| **context** | Additional files loaded into the system prompt |
| **behaviors** | Reusable capability packages (agents + context + tools) |
| **spawn** | Controls what tools sub-agents inherit |

Most bundles don't declare all of these. That's the point — you inherit what you
need from other bundles and add only what's unique.

## How It Works

### Bundle Resolution

When Amplifier loads a bundle, it needs to find it. Bundles are addressable through
three URI formats:

| Format | Example | When to Use |
|--------|---------|-------------|
| **Git URL** | `git+https://github.com/microsoft/amplifier-foundation@main` | Published bundles |
| **Local path** | `./bundles/variant.yaml` or `file:///home/me/my-bundle` | Development and testing |
| **Namespace path** | `recipes:behaviors/recipes` | Referencing within or across registered bundles |

### Namespace Registration

Every bundle declares a name in its frontmatter. That name becomes its **namespace**:

```yaml
bundle:
  name: recipes    # This bundle's namespace is "recipes"
```

Once loaded, you can reference anything inside that bundle using `namespace:path`
syntax. The namespace is always the `bundle.name` value — never the repository name
or git URL.

```
recipes:behaviors/recipes          # behavior within the recipes bundle
recipes:context/instructions.md    # context file
recipes:recipe-author              # an agent definition
foundation:context/shared/base.md  # file in the foundation bundle
```

In Markdown, add `@` to eagerly load a file's contents:
`@foundation:context/shared/common-system-base.md`. In YAML sections, use the bare
`namespace:path` without the `@` prefix.

### Composition Flow

When multiple bundles compose together, Amplifier merges them in order. Later
bundles override earlier ones:

```
                    includes
  ┌──────────────┐ ────────── ┌──────────────┐
  │  foundation   │            │  recipes     │
  │              │            │  (behavior)  │
  │  tools       │            │  tool        │
  │  hooks       │            │  agents      │
  │  orchestrator│            │  context     │
  │  context     │            │              │
  └──────┬───────┘            └──────┬───────┘
         │          merge            │
         └──────────┬────────────────┘
                    ▼
         ┌──────────────────┐
         │  your-bundle.md  │
         │                  │
         │  inherits all    │
         │  from above,     │
         │  adds own        │
         │  instructions    │
         └──────────────────┘
                    │
                    ▼
           Amplifier Session
    (tools + agents + provider + hooks
     + system prompt all assembled)
```

The merge rules follow a predictable pattern:

- **Tools, hooks, providers**: merged by module ID; configs for the same module deep-merge
- **Agents**: merged by name; later definitions win
- **Context**: accumulates with namespace prefixes (no collisions)
- **Markdown body**: replaces entirely (later wins)
- **Session config**: deep-merged (nested dicts merge, scalars override)

## Using Bundles

### Loading a Bundle

> amplifier bundle add git+https://github.com/microsoft/amplifier-bundle-recipes@main

This registers the bundle. Then activate it:

> amplifier bundle use recipes

Or load directly when starting a session:

> amplifier run --bundle ./my-bundle.md "Help me write tests"

### The Ecosystem

Amplifier ships with a rich ecosystem of bundles. Here are the key ones:

| Bundle | What It Provides |
|--------|-----------------|
| **foundation** | The base layer — filesystem, bash, web, search, task delegation, streaming UI |
| **recipes** | Multi-step workflow orchestration via declarative YAML recipes |
| **superpowers** | TDD-driven development modes: brainstorm, plan, execute, verify, finish |
| **python-dev** | Python code quality (ruff, pyright), LSP, and an expert agent |
| **dot-graph** | DOT/Graphviz validation, rendering, and graph intelligence |
| **browser-tester** | Browser automation with operator, researcher, and visual documenter agents |
| **design-intelligence** | 7 specialized agents for design philosophy and knowledge |
| **stories** | Autonomous storytelling with 11 agents and 4 output formats |
| **skills** | Load domain knowledge from curated skill collections |

Most of these follow the same pattern: they include `foundation` and layer on
their own behaviors. You rarely need to understand their internals — just include
them.

## Creating Your Own

### The Thin Bundle Pattern

The most important pattern in bundle design is **thin bundles**: compose from
existing bundles rather than reinventing what they provide.

Here's what NOT to do:

```yaml
# BAD: Redeclares everything foundation already provides
includes:
  - bundle: foundation

tools:                        # foundation already has these!
  - module: tool-filesystem
  - module: tool-bash

session:
  orchestrator:               # foundation already has this!
    module: loop-streaming
```

Here's the right approach:

```yaml
# GOOD: Thin bundle — only declares what's unique
---
bundle:
  name: my-team-assistant
  version: 1.0.0
  description: Our team's development assistant

includes:
  - bundle: git+https://github.com/microsoft/amplifier-foundation@main
  - bundle: my-team-assistant:behaviors/my-team-assistant
---

# Team Assistant

@my-team-assistant:context/instructions.md

---

@foundation:context/shared/common-system-base.md
```

That's the entire bundle. All tools, hooks, orchestrator, and streaming UI come
from foundation. You add only your team's instructions and behaviors.

### The Behavior Pattern

A **behavior** is a reusable capability package that lives in `behaviors/` and can
be included by any bundle. Behaviors bundle together agents, context, and optionally
tools into a composable unit.

Create `behaviors/my-team-assistant.yaml`:

```yaml
bundle:
  name: my-team-assistant-behavior
  version: 1.0.0
  description: Team-specific agents and context

agents:
  include:
    - my-team-assistant:code-reviewer
    - my-team-assistant:test-writer

context:
  include:
    - my-team-assistant:context/instructions.md
```

Then create agent files in `agents/`, context in `context/`, and you have a
reusable capability that any bundle in the ecosystem can include.

### Practical Example: Composing a Custom Bundle

Let's build a bundle for a data science team that needs Python development
tools, recipe workflows, and custom instructions:

```
my-data-science-bundle/
├── bundle.md
├── behaviors/
│   └── data-science.yaml
├── agents/
│   └── notebook-reviewer.md
└── context/
    └── instructions.md
```

**bundle.md** — thin, just includes:

```markdown
---
bundle:
  name: data-science
  version: 1.0.0
  description: Data science team assistant

includes:
  - bundle: git+https://github.com/microsoft/amplifier-foundation@main
  - bundle: git+https://github.com/microsoft/amplifier-bundle-python-dev@main
  - bundle: git+https://github.com/microsoft/amplifier-bundle-recipes@main
  - bundle: data-science:behaviors/data-science
---

# Data Science Assistant

@data-science:context/instructions.md

---

@foundation:context/shared/common-system-base.md
```

**behaviors/data-science.yaml** — the unique value:

```yaml
bundle:
  name: data-science-behavior
  version: 1.0.0

agents:
  include:
    - data-science:notebook-reviewer

context:
  include:
    - data-science:context/instructions.md
```

Three existing bundles composed together, one custom agent added. The team gets
foundation tools, Python linting, recipe workflows, and their own notebook
review agent — all from a bundle.md that's under 20 lines of YAML.

## Bundles vs Modules

This distinction trips people up. Here's the clear separation:

| | Bundle | Module |
|--|--------|--------|
| **What it is** | Configuration package (Markdown + YAML) | Python code package |
| **File format** | `bundle.md` or `.yaml` | Python with `pyproject.toml` |
| **Contains** | Includes, tools, agents, context, instructions | Implementation of one capability |
| **Granularity** | Composes many modules together | Single tool, provider, hook, etc. |
| **Installed via** | `amplifier bundle add <url>` | Referenced from a bundle's `tools:` section |
| **Runs code?** | No — it's pure configuration | Yes — Python with `mount()` entry point |
| **Example** | `amplifier-bundle-recipes` | `amplifier-module-tool-recipes` |
| **Analogy** | A recipe card listing ingredients | One specific ingredient |

A bundle says "I need the recipes tool." A module *is* the recipes tool. Bundles
compose; modules execute.

## Best Practices

**Keep bundles thin.** If you're including foundation, don't redeclare its tools,
hooks, or orchestrator. Your bundle should be a short list of includes plus your
unique instructions.

**Use behaviors for reusability.** Package your agents and context in
`behaviors/` so others can include just your capability without taking your
entire bundle.

**Consolidate instructions.** Put instructions in `context/instructions.md`,
not inline in bundle.md. This lets both your behavior and your root bundle
reference the same content without duplication.

**Don't put secrets in bundles.** Provider API keys, environment-specific paths,
and user preferences belong in `~/.amplifier/settings.yaml`, not in bundle
configuration. This keeps bundles portable across environments.

**Use `@mentions` in Markdown, bare paths in YAML.** The `@` prefix is for
Markdown body text that gets eagerly loaded. YAML sections use `namespace:path`
without `@`. Mixing these up causes silent failures.

**One purpose per bundle.** If your bundle does code review AND deployment AND
monitoring, split it into three bundles with behaviors. Each can be composed
independently.

## Key Takeaways

1. **Bundles are composable configuration** — Markdown files with YAML frontmatter
   that declare what an AI session needs. They are not code.

2. **Composition over duplication** — include foundation and other bundles rather
   than redeclaring their tools and hooks. Follow the thin bundle pattern.

3. **Behaviors are the unit of reuse** — package agents, context, and tools in
   `behaviors/` so any bundle can include your capability.

4. **Three URI formats** address bundles: `git+https://...` for published bundles,
   local paths for development, and `namespace:path` for cross-bundle references.

5. **Bundles compose, modules execute** — bundles are the recipe card listing what
   you need; modules are the implementations that do the work.

6. **The ecosystem is rich** — foundation, recipes, superpowers, python-dev, and
   dozens more are ready to compose into your custom bundle.

7. **Start thin, stay thin** — begin with foundation, add one behavior, write your
   instructions. Expand only when you need to.
