---
id: recipes
type: concepts
title: "Recipes"
---

# Recipes

Recipes are declarative YAML workflows that orchestrate multiple agents across multiple steps. They're perfect for complex, repeatable tasks.

## What is a Recipe?

A recipe defines:

- **Steps** - Sequence of operations
- **Agents** - Which specialist handles each step
- **Context flow** - How results pass between steps
- **Control flow** - Conditions, loops, approval gates

Think of recipes like GitHub Actions for AI agents.

## Why Recipes?

| Ad-hoc Prompts | Recipes |
|----------------|---------|
| One-off conversations | Repeatable workflows |
| Context lost between runs | State persisted |
| Manual orchestration | Automatic step sequencing |
| No checkpoints | Resumable after interruption |
| Implicit flow | Explicit, documented flow |

## Recipe Structure

```yaml
# my-recipe.yaml
name: code-review
description: Comprehensive code review workflow

# Input variables
context:
  file_path: ""  # Required input

# Workflow steps
steps:
  - id: analyze-design
    agent: foundation:zen-architect
    instruction: |
      Analyze {{file_path}} for design issues.
      Check for: complexity, coupling, cohesion.

  - id: check-security
    agent: foundation:security-guardian
    instruction: |
      Review {{file_path}} for security vulnerabilities.
      Check OWASP Top 10.

  - id: create-report
    instruction: |
      Create a code review report combining:
      
      Design Analysis:
      {{analyze-design.result}}
      
      Security Findings:
      {{check-security.result}}
```

## Running Recipes

### Basic Execution

```bash
# Run a recipe
amplifier recipes execute recipes/code-review.yaml \
  --context '{"file_path": "src/auth.py"}'
```

### Interactive Mode

```bash
amplifier

> Run the code-review recipe on src/api/users.py
```

### From the Recipes Bundle

```bash
# First, add the recipes bundle
amplifier bundle add git+https://github.com/microsoft/amplifier-bundle-recipes@main
amplifier bundle use recipes

# See available recipes
amplifier recipes list

# Run an example recipe
amplifier recipes execute examples/code-review.yaml \
  --context '{"file_path": "src/main.py"}'
```

## Context and Variables

### Input Variables

```yaml
context:
  file_path: ""      # Required - no default
  depth: "detailed"  # Optional - has default
```

### Accessing Results

Each step's result is available to subsequent steps:

```yaml
steps:
  - id: step-one
    instruction: "Analyze the code"
    
  - id: step-two
    instruction: |
      Based on this analysis:
      {{step-one.result}}
      
      Suggest improvements.
```

### Template Syntax

```yaml
# Simple variable
{{file_path}}

# Step result
{{step-id.result}}

# Nested access (if result is parsed JSON)
{{step-id.result.summary}}
```

## Approval Gates

Pause for human approval before critical steps:

```yaml
steps:
  - id: plan
    instruction: "Create a migration plan"
    
  - id: execute
    requires_approval: true  # Pauses here
    instruction: "Execute the migration"
```

When execution reaches an approval gate:

```
Step 'execute' requires approval.

To approve: amplifier recipes approve [session-id] execute
To deny: amplifier recipes deny [session-id] execute --reason "needs revision"
```

## Loops and Parallel Execution

### Foreach Loops

```yaml
steps:
  - id: get-files
    instruction: "List all Python files in src/"
    parse_json: true  # Parse result as JSON
    
  - id: review-each
    foreach: "{{get-files.result.files}}"
    instruction: |
      Review {{item}} for issues.
```

### Parallel Execution

```yaml
steps:
  - id: review-files
    foreach: "{{files}}"
    parallel: 3  # Run 3 at a time
    instruction: "Review {{item}}"
```

## Conditional Steps

```yaml
steps:
  - id: check-tests
    instruction: "Are there tests for {{file_path}}?"
    parse_json: true
    
  - id: write-tests
    condition: "{{check-tests.result.has_tests}} == false"
    instruction: "Write tests for {{file_path}}"
```

## Error Handling

```yaml
steps:
  - id: risky-step
    instruction: "Try something that might fail"
    on_error: continue  # or: stop (default)
    
  - id: next-step
    instruction: |
      Previous step status: {{risky-step.status}}
      {{#if risky-step.error}}
      Error was: {{risky-step.error}}
      {{/if}}
```

## Resumable Sessions

Recipes automatically checkpoint progress:

```bash
# If execution is interrupted...
amplifier recipes list  # See active sessions

# Resume where you left off
amplifier recipes resume [session-id]
```

## Example Recipes

### Dependency Upgrade

```yaml
name: dependency-upgrade
description: Safely upgrade project dependencies

steps:
  - id: analyze
    agent: foundation:explorer
    instruction: |
      Analyze dependencies in {{project_path}}.
      Identify outdated packages.

  - id: plan
    agent: foundation:zen-architect
    instruction: |
      Create upgrade plan for:
      {{analyze.result}}
      
      Consider breaking changes.

  - id: approve-plan
    requires_approval: true
    instruction: "Review the upgrade plan"

  - id: execute
    agent: foundation:modular-builder
    instruction: |
      Execute the upgrade plan:
      {{plan.result}}

  - id: test
    instruction: "Run tests to verify upgrades"
```

### Security Audit

```yaml
name: security-audit
description: Comprehensive security review

context:
  target_dir: "src/"

steps:
  - id: scan-deps
    instruction: "Check dependencies for known vulnerabilities"

  - id: review-auth
    agent: foundation:security-guardian
    instruction: "Review authentication code in {{target_dir}}"

  - id: check-secrets
    instruction: "Scan for hardcoded secrets or API keys"

  - id: report
    instruction: |
      Create security report from:
      - Dependencies: {{scan-deps.result}}
      - Auth review: {{review-auth.result}}
      - Secrets scan: {{check-secrets.result}}
```

## Try It Yourself

### Exercise 1: Run an Example Recipe

```bash
# Add recipes bundle
amplifier bundle add git+https://github.com/microsoft/amplifier-bundle-recipes@main

# Run example
amplifier recipes execute examples/code-review.yaml \
  --context '{"file_path": "any-file.py"}'
```

### Exercise 2: Create a Simple Recipe

Create `my-recipe.yaml`:

```yaml
name: explain-and-improve
description: Explain code then suggest improvements

context:
  file: ""

steps:
  - id: explain
    instruction: "Explain what {{file}} does in plain language"
    
  - id: improve
    instruction: |
      Based on this understanding:
      {{explain.result}}
      
      Suggest 3 improvements for {{file}}
```

Run it:
```bash
amplifier recipes execute my-recipe.yaml --context '{"file": "src/main.py"}'
```

## Key Takeaways

1. **Recipes are declarative** - YAML defines the workflow
2. **Context flows forward** - Each step can use previous results
3. **Approval gates add control** - Pause for human review
4. **Recipes are resumable** - Interrupt and continue later
5. **Specialists collaborate** - Different agents for different steps

## Next

Learn about reusable knowledge packages:

→ [Skills](skills.md)
