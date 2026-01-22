---
id: bundles
type: concepts
title: "Understanding Bundles"
---

# Understanding Bundles

Bundles are the fundamental unit of composition in Amplifier. They package capabilities,
behaviors, and configuration into reusable, shareable units that can be combined to create
powerful AI-powered applications.

## What is a Bundle?

A bundle is a composable configuration package that defines what an AI assistant can do.
Think of bundles like LEGO sets—each one provides specific pieces that snap together
with other bundles to build complete solutions.

### Core Characteristics

Bundles share several defining characteristics:

1. **Self-contained**: Each bundle includes everything it needs to function
2. **Composable**: Bundles can include other bundles to build on existing capabilities
3. **Declarative**: Configuration is specified in YAML, not imperative code
4. **Portable**: Bundles can be shared, versioned, and distributed independently

### What Bundles Contain

A typical bundle may include:

- **Configuration**: YAML frontmatter defining metadata and settings
- **Instructions**: Markdown content providing guidance to the AI
- **Behaviors**: References to specific capabilities or tools
- **Context files**: Supporting documentation and examples
- **Includes**: References to other bundles to compose functionality

### The Thin Bundle Pattern

Amplifier follows the "thin bundle" philosophy—bundles should be minimal wrappers
that compose existing capabilities rather than reinventing functionality. A well-designed
bundle:

- References behaviors instead of duplicating logic
- Includes foundation bundles for common capabilities
- Adds only the specific instructions needed for its use case
- Stays focused on a single domain or purpose

## Bundle Structure

Every bundle consists of a markdown file with YAML frontmatter. The frontmatter defines
the bundle's metadata and configuration, while the markdown body provides instructions
and context for the AI.

### Anatomy of a Bundle File

```yaml
---
id: my-bundle
name: My Custom Bundle
version: 1.0.0
description: A bundle that does something useful

# Include other bundles
includes:
  - foundation:core
  - foundation:file-ops

# Reference specific behaviors
behaviors:
  - code-review
  - testing

# Configure providers (optional)
providers:
  primary:
    model: claude-sonnet-4-20250514

# Define custom settings (optional)
settings:
  max_file_size: 10000
  output_format: markdown
---

# My Custom Bundle

Instructions for the AI go here in markdown format.

## What This Bundle Does

Explain the purpose and capabilities...

## How to Use

Provide guidance on usage patterns...
```

### Frontmatter Fields

#### Required Fields

| Field | Description |
|-------|-------------|
| `id` | Unique identifier for the bundle (lowercase, hyphens) |
| `name` | Human-readable display name |

#### Common Optional Fields

| Field | Description |
|-------|-------------|
| `version` | Semantic version string (e.g., "1.0.0") |
| `description` | Brief description of the bundle's purpose |
| `includes` | List of other bundles to include |
| `behaviors` | List of behaviors to enable |
| `providers` | Provider configuration (models, endpoints) |
| `settings` | Custom key-value settings |
| `tools` | Tool configurations and restrictions |
| `context` | Additional context files to load |

### Markdown Body

The markdown body serves as the primary instruction set for the AI. Write clear,
actionable guidance that helps the AI understand:

- What the bundle is designed to do
- How to approach tasks within its domain
- What patterns and practices to follow
- What to avoid or be careful about

## Composition

Composition is where bundles become powerful. Instead of building monolithic
configurations, you compose small, focused bundles into complete solutions.

### The `includes` Field

The `includes` field specifies which bundles to incorporate:

```yaml
includes:
  - foundation:core           # Core capabilities
  - foundation:file-ops       # File operations
  - foundation:git-ops        # Git operations
  - my-org:code-standards     # Organization standards
```

### Include Resolution

Bundles are resolved using a namespace:name pattern:

- `foundation:core` → Built-in foundation bundle
- `my-collection:my-bundle` → Bundle from a custom collection
- `./local-bundle` → Relative path to a local bundle file

### Composition Order

When multiple bundles are composed, they're processed in order:

1. Base bundles (from `includes`) are loaded first
2. Each included bundle's includes are resolved recursively
3. The current bundle's configuration is applied last
4. Later configurations can override earlier ones

### Behaviors

Behaviors are reusable capability definitions that bundles can reference:

```yaml
behaviors:
  - code-review      # Enable code review capabilities
  - testing          # Enable testing capabilities
  - documentation    # Enable documentation generation
```

Behaviors differ from includes in that they represent specific capabilities
rather than complete bundle configurations. A behavior might enable certain
tools, add specific instructions, or configure particular patterns.

### Practical Composition Example

Here's how you might compose a code review bundle:

```yaml
---
id: team-code-review
name: Team Code Review
includes:
  - foundation:core
  - foundation:git-ops
behaviors:
  - code-review
  - security-review
---

# Team Code Review

Use this bundle when reviewing pull requests for our team.

## Review Checklist

1. Check for security vulnerabilities
2. Verify test coverage
3. Ensure code follows our style guide
4. Look for performance issues

## Our Standards

- All functions must have docstrings
- Maximum cyclomatic complexity: 10
- No hardcoded secrets
```

## Creating a Bundle

Follow these steps to create your own bundle.

### Step 1: Define the Purpose

Before writing any configuration, clearly define:

- What problem does this bundle solve?
- Who will use it?
- What capabilities does it need?
- What existing bundles can it build on?

### Step 2: Choose Your Base

Start by identifying which foundation bundles to include:

```yaml
includes:
  - foundation:core    # Almost always needed
```

Add additional foundations based on your needs:

- `foundation:file-ops` for file manipulation
- `foundation:git-ops` for version control
- `foundation:web-research` for internet access

### Step 3: Write the Frontmatter

Create your bundle file with appropriate metadata:

```yaml
---
id: my-project-helper
name: My Project Helper
version: 1.0.0
description: Helps with tasks specific to my project

includes:
  - foundation:core
  - foundation:file-ops

behaviors:
  - testing
---
```

### Step 4: Write Instructions

Add clear, specific instructions in the markdown body:

```markdown
# My Project Helper

You help developers work on the MyProject codebase.

## Project Structure

- `src/` - Source code
- `tests/` - Test files
- `docs/` - Documentation

## Coding Standards

- Use TypeScript strict mode
- Write tests for all public functions
- Follow the error handling patterns in `src/utils/errors.ts`

## Common Tasks

### Adding a New Feature
1. Create a branch from main
2. Implement the feature in `src/features/`
3. Add tests in `tests/features/`
4. Update documentation if needed
```

### Step 5: Test Your Bundle

Test your bundle by running Amplifier with it:

```bash
amp --bundle ./my-bundle.md
```

Verify that:

- All includes resolve correctly
- The AI understands your instructions
- Tools and capabilities work as expected

### Step 6: Iterate and Refine

Based on testing, refine your bundle:

- Add missing instructions
- Remove unnecessary includes
- Clarify ambiguous guidance
- Add examples for complex tasks

## Best Practices

### Keep Bundles Focused

Each bundle should have a clear, singular purpose. If you find yourself adding
unrelated capabilities, consider splitting into multiple bundles.

### Use Semantic Versioning

Version your bundles to track changes:

- **Major**: Breaking changes to behavior or structure
- **Minor**: New capabilities, backward compatible
- **Patch**: Bug fixes and documentation updates

### Document Your Bundles

Include clear documentation in the markdown body:

- What the bundle does
- How to use it effectively
- What it doesn't do (scope boundaries)
- Examples of common tasks

### Leverage Composition

Don't duplicate functionality. If another bundle provides what you need,
include it rather than recreating it.

### Test with Real Scenarios

Validate your bundle with actual use cases, not just theoretical ones.
Real-world testing reveals gaps in instructions and missing capabilities.

## Key Takeaways

1. **Bundles are composable packages** that define AI assistant capabilities through
   declarative YAML configuration and markdown instructions.

2. **Structure matters**: Use frontmatter for configuration (id, includes, behaviors)
   and markdown for instructions and context.

3. **Composition over duplication**: Build on existing bundles using `includes` rather
   than recreating functionality. Follow the thin bundle pattern.

4. **Behaviors extend capabilities**: Reference behaviors to add specific capabilities
   without including full bundle configurations.

5. **Start minimal**: Begin with the smallest set of includes and behaviors needed,
   then add more as requirements emerge.

6. **Instructions are key**: The markdown body is where you differentiate your bundle.
   Write clear, actionable guidance for the AI.

7. **Test iteratively**: Create your bundle, test it with real scenarios, and refine
   based on what you learn.

Bundles embody Amplifier's philosophy of modular, composable design. By creating
well-structured bundles, you build reusable capabilities that can be shared across
projects and teams, enabling consistent AI-powered workflows wherever they're needed.
