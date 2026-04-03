---
id: writing-bundles
type: contributing
title: "Writing a Bundle"
---

# Writing a Bundle

You've been using bundles -- composing foundation with recipes, layering on python-dev.
Now it's time to build one from scratch and publish it. This guide walks through the
file format, composition patterns, testing, and publishing conventions.

## The File Format

A bundle is a Markdown file with YAML frontmatter. The frontmatter declares structure;
the body becomes the system prompt.

```markdown
---
bundle:
  name: my-toolkit
  version: 1.0.0
  description: A custom toolkit for my team

includes:
  - bundle: git+https://github.com/microsoft/amplifier-foundation@main
---

# My Toolkit

Instructions for the AI go here as plain Markdown.
```

That's a complete, functional bundle. Let's break down what each piece does.

### The `bundle:` Section

Every bundle starts with identity:

```yaml
bundle:
  name: my-toolkit        # Becomes the namespace for @mentions
  version: 1.0.0          # Semantic versioning
  description: A custom toolkit for my team
```

The `name` matters -- it becomes the namespace you use in `@my-toolkit:path` references
throughout the ecosystem.

### The `includes:` Section

This is where composition happens. Each entry pulls in another bundle:

```yaml
includes:
  - bundle: git+https://github.com/microsoft/amplifier-foundation@main
  - bundle: git+https://github.com/microsoft/amplifier-bundle-recipes@main
  - bundle: my-toolkit:behaviors/my-toolkit
```

### Other YAML Sections

Bundles can declare any combination of these:

| Section | Purpose |
|---------|---------|
| `tools:` | Tool modules to mount (e.g., `module: tool-bash`) |
| `providers:` | LLM backend configurations |
| `agents:` | Agent references via `include:` |
| `hooks:` | Lifecycle observers (logging, approval, streaming) |
| `context:` | Additional files loaded into the system prompt |
| `behaviors:` | Reusable capability packages (agents + context + tools) |
| `spawn:` | Controls what tools sub-agents inherit |
| `session:` | Orchestrator and session-level configuration |

Most bundles declare very few of these. That's by design.

## The Thin Bundle Pattern

The most important pattern in bundle authoring is **thin bundles**: compose from
existing bundles rather than redeclaring what they already provide.

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

Here's the correct approach:

```markdown
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

All tools, hooks, orchestrator, and streaming UI come from foundation. You add
only what's unique -- your behavior and your instructions.

## Bundles vs Behaviors

This distinction matters for reusability.

A **bundle** is a top-level configuration that defines a complete session. It's what
you pass to `amplifier run --bundle`.

A **behavior** is a reusable capability package that lives in `behaviors/` inside
a bundle. It packages agents, context, and optionally tools into a composable unit
that any bundle can include.

```
my-toolkit/
├── bundle.md                    # The top-level bundle
├── behaviors/
│   └── my-toolkit.yaml          # Reusable capability package
├── agents/
│   ├── code-reviewer.md
│   └── test-writer.md
└── context/
    └── instructions.md
```

The behavior file (`behaviors/my-toolkit.yaml`) wires the pieces together:

```yaml
bundle:
  name: my-toolkit-behavior
  version: 1.0.0
  description: Code review and test writing agents

agents:
  include:
    - my-toolkit:code-reviewer
    - my-toolkit:test-writer

context:
  include:
    - my-toolkit:context/instructions.md
```

Why separate them? Because another bundle can include just your behavior without
taking your entire bundle configuration. Your agents become composable building
blocks for the wider ecosystem.

## Source URI Formats

Bundles are addressable through three URI formats:

| Format | Example | When to Use |
|--------|---------|-------------|
| **Git URL** | `git+https://github.com/microsoft/amplifier-foundation@main` | Published bundles |
| **Local path** | `file:///home/me/my-bundle` or `./bundles/variant.yaml` | Development and testing |
| **Namespace path** | `recipes:behaviors/recipes` | Cross-bundle references |

Git URLs pin to a branch or tag with `@`. Local paths are useful during development
before publishing. Namespace paths reference content within already-loaded bundles
using the `bundle.name` as the prefix.

In Markdown body text, add `@` to eagerly load content:
`@foundation:context/shared/common-system-base.md`. In YAML sections, use the bare
`namespace:path` without the `@` prefix.

## Building a Complete Bundle

Let's walk through creating a data science team bundle step by step.

**1. Create the directory structure:**

```
data-science-bundle/
├── bundle.md
├── behaviors/
│   └── data-science.yaml
├── agents/
│   └── notebook-reviewer.md
└── context/
    └── instructions.md
```

**2. Write `bundle.md`** -- thin, just includes and instructions:

```markdown
---
bundle:
  name: data-science
  version: 1.0.0
  description: Data science team assistant

includes:
  - bundle: git+https://github.com/microsoft/amplifier-foundation@main
  - bundle: git+https://github.com/microsoft/amplifier-bundle-python-dev@main
  - bundle: data-science:behaviors/data-science
---

# Data Science Assistant

@data-science:context/instructions.md

---

@foundation:context/shared/common-system-base.md
```

**3. Write the behavior** (`behaviors/data-science.yaml`):

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

**4. Write the agent** (`agents/notebook-reviewer.md`):

```markdown
---
meta:
  name: notebook-reviewer
  description: "Reviews Jupyter notebooks for reproducibility, code quality,
    and documentation. Use when auditing or cleaning up notebooks."
---

# Notebook Reviewer

You are a notebook review specialist...
```

Three existing bundles composed, one custom agent added, under 20 lines of YAML.

## Testing Your Bundle

Test locally before publishing:

> amplifier run --bundle ./data-science-bundle/bundle.md "What agents are available?"

The `--bundle` flag accepts a local file path. Amplifier resolves the bundle,
loads all includes, and starts a session. Iterate until everything wires up correctly.

Check that your agents are registered:

> amplifier run --bundle ./data-science-bundle/bundle.md "Delegate to notebook-reviewer to review notebooks in examples/"

[Tool: delegate] agent="data-science:notebook-reviewer", instruction="Review notebooks in examples/..."

If the delegation fails, check that the agent is listed in your behavior's
`agents.include` and that the namespace matches your `bundle.name`.

## Publishing Your Bundle

Amplifier bundles follow a naming convention for discoverability:

```
amplifier-bundle-<name>
```

For example: `amplifier-bundle-recipes`, `amplifier-bundle-python-dev`,
`amplifier-bundle-stories`.

To publish:

1. Create a GitHub repository following the naming convention
2. Put your `bundle.md` at the repository root
3. Include `behaviors/`, `agents/`, and `context/` directories as needed
4. Tag releases with semantic versions

Others install your bundle with:

> amplifier bundle add git+https://github.com/your-org/amplifier-bundle-data-science@main

The canonical example to study is `amplifier-bundle-recipes` -- it demonstrates
behaviors, agents, context files, and composition with foundation.

## Best Practices

**Keep bundles thin.** If you're including foundation, don't redeclare its tools
or hooks. Your bundle should be a short list of includes plus your unique content.

**Use behaviors for reusability.** Package agents and context in `behaviors/` so
others can include just your capability independently.

**Consolidate instructions.** Put instructions in `context/instructions.md`, not
inline in bundle.md. This lets both your behavior and root bundle reference the
same content without duplication.

**One purpose per bundle.** If your bundle does code review AND deployment AND
monitoring, split into three bundles with behaviors. Each composes independently.

**Don't put secrets in bundles.** Provider API keys and environment-specific paths
belong in `~/.amplifier/settings.yaml`, not in bundle configuration.

**Use `@mentions` in Markdown, bare paths in YAML.** The `@` prefix is for Markdown
body text. YAML sections use `namespace:path` without `@`. Mixing these up causes
silent failures.
