---
id: tool-recipes
type: tools
title: "Recipes Tool"
---

# Recipes Tool

The Recipes Tool enables declarative, multi-step AI agent workflows through YAML specifications. Instead of manually orchestrating complex tasks, you define what needs to happen and let Amplifier handle the execution, state management, and error recovery.

## What are Recipes?

Recipes are **declarative YAML workflows** that define multi-step agent tasks. They bring structure and repeatability to complex operations that would otherwise require multiple manual prompts.

Key characteristics:

- **Declarative**: You specify *what* to do, not *how* to do it
- **Sequential execution**: Steps run in order with state persistence
- **Agent delegation**: Each step can use different specialized agents
- **Context accumulation**: Results from earlier steps flow to later ones
- **Automatic checkpointing**: Sessions can be resumed if interrupted
- **Error handling**: Built-in retry logic and failure recovery

Think of recipes as "saved workflows" - instead of repeating the same multi-step process manually, you encode it once and run it whenever needed.

## Basic Recipe Structure

A minimal recipe looks like this:

```yaml
name: code-review
description: Review code changes and suggest improvements

steps:
  - id: analyze
    agent: foundation:zen-architect
    prompt: |
      Analyze the code in {{ file_path }} for:
      - Code quality issues
      - Potential bugs
      - Performance concerns

  - id: suggest
    agent: foundation:modular-builder
    prompt: |
      Based on this analysis:
      {{ steps.analyze.result }}
      
      Suggest specific improvements with code examples.
```

## Executing Recipes

### Basic Execution

Run a recipe from the command line:

```bash
amplifier run "execute recipe.yaml"
```

Or use the recipes tool directly in a session:

```
Execute the code-review recipe at ./recipes/code-review.yaml
```

### Passing Context

Recipes often need input parameters. Pass context variables when executing:

```bash
amplifier run "execute recipe.yaml with file_path=src/auth.py"
```

Multiple context values:

```bash
amplifier run "execute recipe.yaml with file_path=src/auth.py, depth=detailed"
```

Context variables are available in prompts using `{{ variable_name }}` syntax.

### From Within Amplifier

You can also execute recipes programmatically:

```
Use the recipes tool to execute ./my-recipe.yaml with context:
- project_name: my-app
- environment: staging
```

## Operations Reference

| Operation | Purpose | Required Parameters |
|-----------|---------|---------------------|
| `execute` | Run a recipe from YAML file | `recipe_path`, optional `context` |
| `resume` | Continue an interrupted session | `session_id` |
| `list` | List all active recipe sessions | None |
| `validate` | Check recipe YAML structure | `recipe_path` |
| `approvals` | List pending approval gates | None |
| `approve` | Approve a stage to continue | `session_id`, `stage_name` |
| `deny` | Deny a stage and stop execution | `session_id`, `stage_name` |

### Operation Examples

**Validate before running:**
```
Validate the recipe at ./recipes/deploy.yaml
```

**List active sessions:**
```
List all active recipe sessions
```

**Resume an interrupted session:**
```
Resume recipe session recipe_20260111_143022_a3f2
```

## Approval Gates

For workflows requiring human oversight, recipes support **staged execution** with approval gates. The recipe pauses at designated points and waits for explicit approval before continuing.

### Defining Stages

```yaml
name: production-deploy
description: Deploy to production with safety gates

stages:
  - name: planning
    steps:
      - id: plan
        agent: foundation:zen-architect
        prompt: Create deployment plan for {{ service_name }}

  - name: execution
    requires_approval: true
    steps:
      - id: deploy
        agent: foundation:modular-builder
        prompt: |
          Execute deployment based on:
          {{ stages.planning.steps.plan.result }}
```

### Managing Approvals

**List pending approvals:**
```
Show all pending recipe approvals
```

**Approve a stage:**
```
Approve the 'execution' stage for session recipe_20260111_143022_a3f2
```

**Deny with reason:**
```
Deny the 'execution' stage for session recipe_20260111_143022_a3f2 
because the deployment plan needs revision
```

## Advanced Features

### Conditional Execution

Steps can include conditions:

```yaml
steps:
  - id: run-tests
    agent: foundation:test-coverage
    prompt: Run the test suite
    
  - id: deploy
    agent: foundation:modular-builder
    condition: "{{ steps.run-tests.success }}"
    prompt: Deploy the application
```

### Error Handling

Configure retry behavior and error strategies:

```yaml
steps:
  - id: api-call
    agent: foundation:integration-specialist
    prompt: Call the external API
    on_error: retry
    max_retries: 3
    retry_delay: 5s
```

### Foreach Loops

Process multiple items:

```yaml
steps:
  - id: review-files
    foreach: "{{ files }}"
    as: file
    agent: foundation:zen-architect
    prompt: Review {{ file }} for issues
```

### Timeouts

Set execution limits:

```yaml
steps:
  - id: long-task
    agent: foundation:modular-builder
    prompt: Process the large dataset
    timeout: 30m
```

## Best Practices

### 1. Keep Steps Focused

Each step should have a single, clear purpose:

```yaml
# Good: Focused steps
steps:
  - id: analyze
    prompt: Analyze the codebase structure
  - id: identify-issues
    prompt: Identify potential issues from analysis
  - id: suggest-fixes
    prompt: Suggest fixes for identified issues

# Avoid: Overloaded steps
steps:
  - id: do-everything
    prompt: Analyze code, find issues, and fix them all at once
```

### 2. Use Descriptive IDs

Step IDs should clearly indicate their purpose:

```yaml
# Good
- id: validate-input
- id: generate-report
- id: notify-team

# Avoid
- id: step1
- id: process
- id: final
```

### 3. Leverage Context Accumulation

Reference previous step results to build context:

```yaml
steps:
  - id: gather-requirements
    prompt: List all requirements from {{ spec_file }}
    
  - id: design-solution
    prompt: |
      Design a solution for these requirements:
      {{ steps.gather-requirements.result }}
```

### 4. Add Approval Gates for Critical Operations

Any step that modifies production systems should have an approval gate:

```yaml
stages:
  - name: prepare
    steps: [...]
    
  - name: deploy-production
    requires_approval: true
    steps:
      - id: deploy
        prompt: Deploy to production
```

### 5. Validate Before Executing

Always validate new or modified recipes:

```bash
amplifier run "validate my-recipe.yaml"
```

### 6. Use Appropriate Agents

Match agents to tasks:

- `foundation:zen-architect` - Planning and analysis
- `foundation:modular-builder` - Implementation
- `foundation:test-coverage` - Testing
- `foundation:security-guardian` - Security reviews

## Try It Yourself

### Exercise 1: Create a Simple Recipe

Create `hello-recipe.yaml`:

```yaml
name: hello-world
description: A simple introduction to recipes

steps:
  - id: greet
    agent: foundation:explorer
    prompt: |
      List the files in the current directory and 
      describe what this project appears to be about.
```

Run it:
```bash
amplifier run "execute hello-recipe.yaml"
```

### Exercise 2: Recipe with Context

Create `file-analyzer.yaml`:

```yaml
name: file-analyzer
description: Analyze a specific file

steps:
  - id: read-file
    agent: foundation:file-ops
    prompt: Read and summarize {{ target_file }}
    
  - id: suggest-improvements
    agent: foundation:zen-architect
    prompt: |
      Based on this file content:
      {{ steps.read-file.result }}
      
      Suggest three improvements.
```

Run with context:
```bash
amplifier run "execute file-analyzer.yaml with target_file=README.md"
```

### Exercise 3: Multi-Stage with Approval

Create `safe-refactor.yaml`:

```yaml
name: safe-refactor
description: Refactor with human approval

stages:
  - name: analysis
    steps:
      - id: analyze
        agent: foundation:zen-architect
        prompt: Analyze {{ file_path }} and propose refactoring

  - name: implementation
    requires_approval: true
    steps:
      - id: refactor
        agent: foundation:modular-builder
        prompt: |
          Implement the refactoring plan:
          {{ stages.analysis.steps.analyze.result }}
```

This recipe will pause after analysis, letting you review the plan before any changes are made.

## Summary

The Recipes Tool transforms complex, multi-step workflows into repeatable, reliable automation:

- **Define once, run many times** - Encode workflows as YAML
- **Built-in safety** - Approval gates for critical operations
- **Resilient execution** - Automatic checkpointing and resume
- **Context-aware** - Steps build on previous results

Start with simple recipes and gradually add complexity as you become comfortable with the patterns.
