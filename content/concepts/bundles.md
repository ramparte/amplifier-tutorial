---
id: bundles
type: concepts
title: "Understanding Bundles"
description: "Learn how bundles package agent capabilities into composable, reusable configurations"
prerequisites:
  - concepts/agents
  - concepts/tools
related:
  - concepts/behaviors
  - concepts/collections
  - guides/creating-bundles
---

# Understanding Bundles

Bundles are the primary unit of composition in Amplifier. They package together everything an agent needs—instructions, tools, behaviors, and context—into a single, reusable configuration that can be shared, combined, and customized.

## What is a Bundle?

A bundle is a composable configuration package that defines an agent's capabilities. Think of it like a recipe: it specifies the ingredients (tools, context files), the instructions (system prompts, behaviors), and how they combine to create a specific agent personality and capability set.

### The Core Idea

Traditional AI agent configuration often involves:
- Scattered configuration files
- Duplicated prompts across projects
- Manual tool registration
- Inconsistent setups between environments

Bundles solve this by providing a **single source of truth** for an agent's configuration that can be:

- **Versioned** alongside your code
- **Shared** across teams and projects
- **Composed** to build complex agents from simple parts
- **Overridden** for environment-specific customization

### Bundle vs Agent

It's important to understand the distinction:

| Aspect | Bundle | Agent |
|--------|--------|-------|
| Definition | Configuration package | Runtime instance |
| Contains | Instructions, tools, behaviors | Active session state |
| Lifetime | Persistent (file-based) | Ephemeral (session-based) |
| Purpose | Define capabilities | Execute tasks |

A bundle is the *blueprint*; an agent is the *running instance* created from that blueprint.

## Bundle Structure

Every bundle follows a standard structure using Markdown files with YAML frontmatter. This format is both human-readable and machine-parseable.

### Basic Bundle Layout

```
my-bundle/
├── bundle.md           # Main bundle definition
├── context/            # Supporting context files
│   ├── guidelines.md
│   └── examples.md
├── behaviors/          # Custom behaviors
│   └── review-code.md
└── tools/              # Custom tool definitions (optional)
    └── lint.yaml
```

### The bundle.md File

The `bundle.md` file is the entry point for every bundle. It contains:

```markdown
---
name: code-reviewer
description: "Expert code review agent"
version: "1.0.0"

# Tool configuration
tools:
  - read_file
  - write_file
  - grep
  - glob
  - bash

# Include other bundles
includes:
  - foundation:core-tools
  - my-org:coding-standards

# Behaviors to activate
behaviors:
  - thorough-review
  - security-focus

# Context files to load
context:
  - context/guidelines.md
  - context/examples.md
---

# Code Reviewer

You are an expert code reviewer focused on quality, security, and maintainability.

## Your Approach

1. Read the code thoroughly before commenting
2. Focus on substantive issues, not style nitpicks
3. Provide actionable suggestions with examples
4. Acknowledge good patterns when you see them

## Review Checklist

- [ ] Logic correctness
- [ ] Error handling
- [ ] Security considerations
- [ ] Performance implications
- [ ] Test coverage
```

### Frontmatter Fields

| Field | Required | Description |
|-------|----------|-------------|
| `name` | Yes | Unique identifier for the bundle |
| `description` | No | Human-readable description |
| `version` | No | Semantic version string |
| `tools` | No | List of tools to enable |
| `includes` | No | Other bundles to compose |
| `behaviors` | No | Behaviors to activate |
| `context` | No | Additional context files to load |

### Body Content

The markdown body after the frontmatter becomes the agent's system prompt. This is where you define:

- The agent's role and personality
- Specific instructions and guidelines
- Domain knowledge
- Response formats

## Composition

One of the most powerful features of bundles is composition. You can build complex agents by combining simpler bundles.

### The includes Field

Use `includes` to inherit from other bundles:

```yaml
includes:
  - foundation:core-tools      # From foundation collection
  - my-org:coding-standards    # From your organization
  - ./local-overrides          # From relative path
```

When bundles are composed:
1. Tools are **merged** (union of all tools)
2. Context is **accumulated** (all context files load)
3. Behaviors are **combined** (all behaviors activate)
4. Instructions are **layered** (child overrides parent)

### Composition Order Matters

Bundles are processed in order, with later bundles taking precedence:

```yaml
includes:
  - base-agent           # Loaded first (lowest priority)
  - security-focused     # Adds security behaviors
  - my-customizations    # Loaded last (highest priority)
```

### Behaviors

Behaviors are reusable instruction fragments that modify agent behavior. Unlike full bundles, behaviors are lightweight and focused on a single concern.

```markdown
---
name: security-focus
trigger: "when reviewing code"
---

## Security Review Requirements

When reviewing code, always check for:
- Input validation and sanitization
- Authentication and authorization
- Sensitive data exposure
- SQL injection vulnerabilities
- XSS attack vectors
```

Behaviors can be:
- **Always active**: Load with every request
- **Conditionally triggered**: Activate based on context
- **User-invoked**: Activated by explicit command

### Practical Composition Example

Here's how you might compose a specialized agent:

```yaml
# devops-agent/bundle.md
---
name: devops-agent
includes:
  - foundation:core-tools
  - foundation:git-ops
  - cloud:aws-tools
behaviors:
  - infrastructure-as-code
  - security-compliance
  - cost-awareness
---

# DevOps Agent

You specialize in infrastructure, deployment, and operational concerns.
```

This agent inherits:
- File and shell tools from `core-tools`
- Git capabilities from `git-ops`  
- AWS-specific tools from `aws-tools`
- Three domain-specific behaviors

## Creating a Bundle

Let's walk through creating a bundle from scratch.

### Step 1: Define the Purpose

Before writing any configuration, clearly define:
- What problem does this agent solve?
- What tools does it need?
- What domain knowledge is required?
- Who will use it?

### Step 2: Create the Structure

```bash
mkdir -p my-agent/{context,behaviors}
touch my-agent/bundle.md
```

### Step 3: Write the Bundle Definition

```markdown
---
name: documentation-writer
description: "Creates clear, comprehensive documentation"
version: "1.0.0"

tools:
  - read_file
  - write_file
  - glob
  - grep

includes:
  - foundation:core-tools

context:
  - context/style-guide.md
---

# Documentation Writer

You are an expert technical writer who creates clear, 
comprehensive documentation.

## Writing Principles

1. **Clarity first**: Use simple language
2. **Structure matters**: Organize logically
3. **Examples help**: Show, don't just tell
4. **Audience awareness**: Know your reader
```

### Step 4: Add Context Files

Create supporting context that provides domain knowledge:

```markdown
<!-- context/style-guide.md -->
# Documentation Style Guide

## Voice and Tone
- Use active voice
- Be direct and concise
- Avoid jargon unless necessary

## Formatting
- Use headers to organize content
- Include code examples in fenced blocks
- Add tables for comparative information
```

### Step 5: Test the Bundle

Run your bundle and verify it works as expected:

```bash
amp --bundle ./my-agent
```

### Bundle Best Practices

**DO:**
- Keep bundles focused on a single domain
- Use composition instead of duplication
- Version your bundles alongside code
- Document the bundle's purpose and usage

**DON'T:**
- Create monolithic bundles that do everything
- Duplicate instructions across bundles
- Hardcode environment-specific values
- Include sensitive data in bundles

## Advanced Topics

### Environment-Specific Overrides

Use includes with environment bundles:

```yaml
includes:
  - base-agent
  - ${ENVIRONMENT:-development}  # Loads dev/staging/prod bundle
```

### Bundle Inheritance Patterns

**Specialization**: Start general, get specific
```
foundation:core → domain:backend → project:api-service
```

**Layering**: Add capabilities incrementally
```
base + security + monitoring + custom
```

### Collection Organization

Group related bundles into collections:

```
my-collection/
├── collection.yaml     # Collection metadata
├── agents/
│   ├── reviewer/
│   └── writer/
├── behaviors/
│   └── shared/
└── context/
    └── common/
```

## Key Takeaways

1. **Bundles are composable packages** that define everything an agent needs—tools, instructions, behaviors, and context.

2. **Markdown + YAML frontmatter** provides a human-readable, version-controllable format for agent configuration.

3. **Composition is powerful**: Use `includes` to build complex agents from simple, focused bundles. Tools merge, context accumulates, and instructions layer.

4. **Behaviors add modularity**: Extract reusable instruction fragments into behaviors that can be mixed and matched.

5. **Structure matters**: Follow the standard bundle layout for consistency and discoverability.

6. **Keep bundles focused**: One bundle should solve one class of problems. Use composition to combine capabilities.

7. **Test and iterate**: Bundles are code—version them, test them, and refine them based on real usage.

---

## Next Steps

- [Creating Your First Bundle](../guides/creating-bundles.md) - Hands-on tutorial
- [Behaviors Deep Dive](./behaviors.md) - Learn about behavior composition
- [Collections](./collections.md) - Organizing bundles at scale
