---
id: routing
type: concept
title: "Model Routing"
---

# Model Routing

Every task has an ideal model. A quick file rename doesn't need the same
horsepower as a multi-step architectural redesign. Model routing is the system
that automatically matches each task to the right model -- so you get speed
where speed matters and depth where depth matters, without thinking about it.

## The Routing Matrix

At the heart of model routing is the **routing matrix** -- a configuration that
maps abstract *roles* to concrete *models*. You never hard-code "use GPT-4" or
"use Claude Sonnet." Instead, you declare what *kind* of work you're doing, and
the matrix resolves it to the best available model.

```
You: "Rename all .jsx files to .tsx"
  [role: fast]  →  matrix lookup  →  Flash-tier model  →  done in 2s

You: "Design a plugin architecture for this auth system"
  [role: reasoning]  →  matrix lookup  →  Heavy-tier, extra-high reasoning  →  deep analysis
```

This separation means you can swap models, upgrade providers, or tune cost
profiles -- all without changing a single agent definition.

## The 13 Roles

Amplifier defines 13 model roles organized into five categories. Each role is a
semantic declaration of intent, not a model name.

### Foundation

| Role | Personality | Use When |
|------|------------|----------|
| `general` | The explorer -- versatile, no specialization needed | Default catch-all; conversation, planning, coordination |
| `fast` | The file clerk -- quick utility tasks | Parsing, classification, file ops, bulk transforms |

### Coding

| Role | Personality | Use When |
|------|------------|----------|
| `coding` | The bug hunter -- implementation specialist | Code generation, debugging, refactoring |
| `ui-coding` | The pixel whisperer -- spatial reasoning | Frontend components, layouts, styling, CSS |
| `security-audit` | The paranoid reviewer -- attack surface analysis | Vulnerability assessment, code auditing, threat modeling |

### Cognitive

| Role | Personality | Use When |
|------|------------|----------|
| `reasoning` | The zen architect -- deep multi-step analysis | System design, complex tradeoffs, architectural decisions |
| `critique` | The devil's advocate -- finding flaws | Evaluating existing work, reviewing plans, poking holes |
| `creative` | The art director -- aesthetic judgment | Design direction, creative output, visual concepts |
| `writing` | The storyteller -- long-form content | Documentation, marketing copy, case studies, tutorials |
| `research` | The detective -- deep investigation | Information synthesis, multi-source analysis, fact-finding |

### Capability

| Role | Personality | Use When |
|------|------------|----------|
| `vision` | The observer -- understanding visual input | Screenshots, diagrams, UI mockups, image analysis |
| `image-gen` | The illustrator -- creating visual output | Image generation, mockups, visual ideation |

### Operational

| Role | Personality | Use When |
|------|------------|----------|
| `critical-ops` | The mission controller -- high-reliability tasks | Infrastructure changes, orchestration, coordination |

## Fallback Chains

Not every model supports every role. When the matrix can't find a model for your
requested role, it walks a **fallback chain** -- trying each role left-to-right
until one resolves.

```
Requested: ui-coding
Chain:     [ui-coding, coding, general]

Step 1: ui-coding  →  no model configured?  →  try next
Step 2: coding     →  found! Use this model
```

The chains are designed so specialized roles degrade gracefully into broader
ones:

| Starting Role | Fallback Chain |
|--------------|----------------|
| `ui-coding` | ui-coding → coding → general |
| `security-audit` | security-audit → coding → general |
| `critique` | critique → reasoning → general |
| `creative` | creative → writing → general |
| `research` | research → reasoning → general |
| `image-gen` | image-gen *(no fallback -- capability required)* |
| `vision` | vision *(no fallback -- capability required)* |
| `general` | general *(terminal -- always configured)* |

Notice that capability roles like `vision` and `image-gen` have no fallback.
You can't approximate image generation with a text model. If the capability
isn't available, the request fails explicitly rather than silently degrading.

## Using Roles in Agents

You declare what role an agent needs in its frontmatter with `model_role`:

```markdown
---
meta:
  name: code-reviewer
  description: "Reviews code for quality and security issues."
model_role: critique
---

# Code Reviewer

You are a code review specialist. Analyze the code for...
```

When this agent runs, the routing matrix resolves `critique` to whatever model
is configured for analytical evaluation in the active matrix.

### In Delegate Calls

When Amplifier delegates to an agent, the role travels with the delegation:

> Review auth.py for security issues and suggest hardening measures.

[Tool: delegate] agent="foundation:code-reviewer", instruction="Review src/auth.py for security vulnerabilities, injection risks, and authentication bypasses."

The delegate tool reads `model_role: critique` from the agent's frontmatter and
routes the sub-session to the appropriate model. You don't specify the model --
the agent's definition carries its own routing preference.

## Matrix Configuration

The routing matrix is configured in your bundle's YAML or at the global settings
level. Amplifier ships with a `balanced` matrix as the default:

```yaml
routing:
  matrix: balanced
```

Matrices define which model tier handles each role:

```
Matrix: balanced
┌─────────────────┬──────────┬───────────┐
│ Role            │ Tier     │ Reasoning │
├─────────────────┼──────────┼───────────┤
│ general         │ Mid      │ default   │
│ fast            │ Flash    │ default   │
│ coding          │ Mid      │ high      │
│ ui-coding       │ Mid      │ high      │
│ security-audit  │ Heavy    │ high      │
│ reasoning       │ Heavy    │ extra-high│
│ critique        │ Heavy    │ high      │
│ creative        │ Mid      │ high      │
│ writing         │ Mid      │ default   │
│ research        │ Heavy    │ high      │
│ vision          │ Mid      │ default   │
│ image-gen       │ Mid      │ default   │
│ critical-ops    │ Heavy    │ extra-high│
└─────────────────┴──────────┴───────────┘
```

### Model Tiers

The matrix uses abstract **tiers** rather than specific model names, making
matrices portable across providers:

| Tier | Characteristic | Example Models |
|------|---------------|----------------|
| **Heavy** | Maximum capability, highest cost | GPT-4, Claude Opus, Gemini Ultra |
| **Mid** | Strong general-purpose, balanced cost | Claude Sonnet, GPT-4o, Gemini Pro |
| **Flash** | Fast and cheap, good for simple tasks | Claude Haiku, GPT-4o-mini, Gemini Flash |

### Reasoning Effort

Each role also specifies a **reasoning effort** level -- `default`, `high`, or
`extra-high` -- controlling how much "thinking" the model does. Combined with
tiers, this creates a 3x3 grid of cost and capability:

```
                 default      high         extra-high
  Heavy      │  capable    │ deep        │ maximum     │
  Mid        │  balanced   │ thorough    │ very deep   │
  Flash      │  quick      │ careful     │ reasoned    │
```

## Choosing the Right Role

When you're building an agent or choosing how to delegate, use this decision
flowchart:

```
Is it a simple utility task (rename, parse, classify)?
  → fast

Does it involve writing or modifying code?
  → coding (or ui-coding for frontend, security-audit for security)

Does it require analyzing something that already exists?
  → critique

Does it require designing something new and complex?
  → reasoning

Does it require creative or aesthetic judgment?
  → creative

Does it require long-form prose?
  → writing

Does it require synthesizing information from many sources?
  → research

Does it involve images (input or output)?
  → vision (input) or image-gen (output)

Is it a high-stakes operational task?
  → critical-ops

None of the above?
  → general
```

The key principle: **declare intent, not implementation.** Say *what kind of
thinking* the task requires, and let the routing matrix handle the rest. When
better models arrive or costs change, your agents keep working -- only the
matrix needs updating.

## What You've Learned

- The routing matrix maps 13 semantic roles to concrete models via abstract tiers
- Roles are organized into five categories: Foundation, Coding, Cognitive,
  Capability, and Operational
- Fallback chains degrade gracefully when a role isn't configured
- Agents declare their role in frontmatter with `model_role`
- Model tiers (Heavy/Mid/Flash) combined with reasoning effort
  (default/high/extra-high) create a cost-capability grid
- You declare intent, not implementation -- the matrix handles resolution
