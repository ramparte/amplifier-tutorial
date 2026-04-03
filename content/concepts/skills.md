---
id: skills
type: concept
title: "Skills"
---

# Skills

Every AI agent starts with general knowledge. Skills make it specific. When you
load a skill, you inject domain expertise — best practices, decision frameworks,
workflows — directly into the agent's context. The agent doesn't just know
*about* Docker or Playwright; it knows *how your team uses* them.

## What is a Skill?

A skill is a **loadable knowledge package** — a markdown file with YAML frontmatter
that follows the [Agent Skills specification](https://agentskills.io). Think of it
as a reference manual the agent consults on demand, not a tool it executes.

Unlike tools (which *do* things) and agents (which *delegate* things), skills
*teach* things. They shape how the agent approaches problems.

```markdown
---
name: react-testing
description: >
  Use when writing or reviewing React component tests,
  especially with async state changes or user interactions
---

# React Testing Patterns

## When to Use
- Writing tests for React components with state
- Testing async operations (API calls, timers)
- Debugging flaky test suites

## Core Pattern: Async State Changes
Always use `waitFor` instead of manual timeouts...
```

That's a complete skill. A markdown file. No code, no build step, no registration.

## How It Works

### Three-Level Progressive Disclosure

Skills use a token-efficient loading strategy. The agent peels back layers only
as needed:

| Level | What Loads | Token Cost | When |
|-------|-----------|------------|------|
| **1 — Metadata** | Name + description | ~100 tokens | Always visible via list/search |
| **2 — Content** | Full markdown body | ~1–5k tokens | Loaded on demand via `load_skill` |
| **3 — References** | Companion files | 0 until accessed | Read via `skill_directory` path |

A simple task ("take a screenshot") might need only Level 1–2. A complex task
("set up Playwright with auth state and parallel contexts") pulls Level 3 too.

### The Visibility Hook

Before each request, Amplifier shows the agent a list of available skills —
Level 1 metadata. The agent sees what expertise is available without consuming
context. When a skill looks relevant, the agent loads it. That "Available
skills" list in your session? That's the visibility hook at work.

## Using Skills

The `load_skill` tool has four operations that mirror the progressive levels:

> List all available skills.

```
[Tool: load_skill] list=True
→ 23 skills found: brainstorming, systematic-debugging, react-testing...
```

> Search for skills related to testing.

```
[Tool: load_skill] search="testing"
→ react-testing, integration-testing-discipline, test-driven-development
```

> What does react-testing cover?

```
[Tool: load_skill] info="react-testing"
→ name: react-testing, version: 1.0.0
  description: Use when writing or reviewing React component tests...
```

The `info` operation gives you metadata without loading content — check
relevance before committing context tokens.

> Load the react-testing skill and help me write tests for this form.

```
[Tool: load_skill] skill_name="react-testing"
→ Skill loaded (2,340 tokens). skill_directory: /path/to/react-testing/
```

Now the agent has the full skill in context. Its next response follows your
team's patterns instead of generic advice. And if the skill includes companion
files (examples, templates), the agent reads them via the returned
`skill_directory` — Level 3, zero tokens until accessed.

### Where Skills Come From

Skills are discovered from multiple directories, in priority order:

| Location | Scope | Typical Use |
|----------|-------|-------------|
| `.amplifier/skills/` | Workspace | Project-specific patterns |
| `~/.amplifier/skills/` | User | Personal workflows across projects |
| Bundle skill sources | Distribution | Skills shipped with bundles |

**First-match-wins**: workspace copies override user copies, letting you
customize shared skills per-project.

## Creating Your Own

A skill is a directory with at least a `SKILL.md` file:

```
docker-multi-stage/
├── SKILL.md           # Required: main content
├── examples/          # Optional: companion files
│   └── basic.dockerfile
└── templates/         # Optional: reusable templates
```

### Writing the SKILL.md

````markdown
---
name: docker-multi-stage
description: >
  Use when building Dockerfiles, optimizing image sizes,
  or debugging container build failures
---

# Docker Multi-Stage Builds

## Overview
Separate build dependencies from runtime for minimal production images.

## Core Pattern

```dockerfile
FROM node:20 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci && npm run build

FROM node:20-slim
COPY --from=builder /app/dist ./dist
CMD ["node", "dist/index.js"]
```

## Common Mistakes
- Installing dev dependencies in production stage
- Using `latest` tag (breaks reproducibility)
- Running as root in production containers
````

Place it in `.amplifier/skills/docker-multi-stage/SKILL.md` for workspace scope
or `~/.amplifier/skills/docker-multi-stage/SKILL.md` for personal use.

### Descriptions Are Triggers

The description field determines whether the agent loads your skill. Write it
as a trigger condition, not a summary:

```yaml
# Good: tells the agent WHEN to load
description: Use when building Dockerfiles or debugging build failures

# Bad: summarizes WHAT the skill does
description: Teaches multi-stage Docker builds with layer caching
```

Start with "Use when..." and focus on symptoms and situations. If the
description summarizes the workflow, the agent may follow the summary instead
of reading the full skill.

## Enhanced Features

### Fork Execution

Skills with `context: fork` run as isolated subagents rather than loading into
your context:

```yaml
---
name: skills-assist
description: Authoritative consultant for skills-related questions
context: fork
---
```

Fork skills are useful for heavyweight expertise that would consume too much
of your main context, or skills that need conversational back-and-forth.

### User-Invocable Skills (Power Skills)

Skills with `disable-model-invocation: true` are triggered by you via slash
commands, not loaded by the agent automatically:

```yaml
---
name: code-review
description: Review changed code for quality and efficiency
disable-model-invocation: true
---
```

Invoke these with `/code-review`, `/mass-change`, `/session-debug`. They're
power skills for workflows you initiate deliberately.

### Skill Sources in Bundles

Bundles can register skill directories so their skills appear in discovery:

```yaml
bundle:
  name: my-team-bundle
  version: 1.0.0

skill_sources:
  - path: ./skills
```

## Skills vs Agents

This is the decision that trips people up:

| | Skill | Agent |
|--|-------|-------|
| **Provides** | Knowledge for the current agent | Independent worker with own context |
| **Execution** | Loads into your session | Spawns as separate sub-session |
| **Context** | Shares your context window | Has its own context window |
| **Interaction** | Shapes ongoing conversation | One-shot task → report |
| **Best for** | Patterns, standards, frameworks | Delegation, parallel work, isolation |

**Use a skill when** the agent needs to know *how* to approach something —
testing patterns, coding standards, debugging frameworks. The knowledge stays
in context and influences every subsequent response.

**Use an agent when** you need to *delegate* work — run an analysis, generate
a report, perform a task in isolation.

**Fork skills bridge the gap** — discovered through the skill system but
running as isolated subagents, they combine skill discovery with agent isolation.

## Best Practices

**Load on demand, not preemptively.** Each loaded skill consumes context tokens.
Use `info` to check relevance first, and load only what the current task needs.

**Keep skills focused.** One domain per skill. "React Testing" is good;
"Frontend Development" is too broad. Focused skills are easier to discover and
provide more actionable guidance.

**Write for triggers, not summaries.** Your description answers "when should I
load this?" not "what does this contain?" Start with "Use when..."

**Use companion files for heavy reference.** Keep SKILL.md concise (under 5k
tokens). Move lengthy API references, examples, and templates into companion
files that load only when needed.

**Workspace overrides user.** Keep personal defaults in `~/.amplifier/skills/`
and project-specific overrides in `.amplifier/skills/`.

## Key Takeaways

1. **Skills are loadable expertise** — markdown files that inject domain
   knowledge into agents on demand, following the Agent Skills specification.

2. **Three levels of disclosure** — metadata (~100 tokens) is always visible,
   content (~1–5k tokens) loads on demand, companions load only when accessed.

3. **Discovery before loading** — use `list`, `search`, and `info` to find
   the right skill without spending context tokens.

4. **Location determines scope** — workspace skills override user skills,
   enabling project-specific customization with personal defaults as fallback.

5. **Skills teach, agents do** — use skills for knowledge and patterns, agents
   for delegation and parallel work. Fork skills bridge the gap.

6. **Descriptions are triggers** — write "Use when..." conditions, not workflow
   summaries. The description determines whether the agent loads the skill.

---

## Related Concepts

- [Agents](./agents.md) — Independent workers for task delegation
- [Bundles](./bundles.md) — Packaging skills with tools, agents, and configuration
- [Architecture](./architecture.md) — How skills fit into the Amplifier system
