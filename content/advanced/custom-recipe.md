---
id: custom-recipe
type: advanced
title: "Creating Custom Recipes"
---

# Creating Custom Recipes

Build multi-step workflows that orchestrate agents.

## Overview

Recipes let you:

- **Orchestrate agents** - Chain specialists together
- **Create repeatable workflows** - Run the same process consistently
- **Add approval gates** - Human-in-the-loop checkpoints
- **Handle complex flows** - Loops, conditions, parallel execution

## Recipe Structure

```yaml
# my-recipe.yaml
name: my-workflow
description: What this recipe does

# Input variables
context:
  input_var: ""       # Required (no default)
  optional_var: "default"  # Optional (has default)

# Workflow steps
steps:
  - id: step-one
    instruction: "Do the first thing with {{input_var}}"
    
  - id: step-two
    agent: foundation:zen-architect
    instruction: |
      Based on: {{step-one.result}}
      Do the second thing.
```

## Basic Recipe

### Simple Two-Step Recipe

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

### Run It

```bash
amp recipes execute explain-and-improve.yaml \
  --context '{"file": "src/auth.py"}'
```

## Using Agents

### Specify Agent for Step

```yaml
steps:
  - id: design
    agent: foundation:zen-architect
    instruction: "Design a solution for {{problem}}"
    
  - id: implement
    agent: foundation:modular-builder
    instruction: |
      Implement the design:
      {{design.result}}
    
  - id: review
    agent: foundation:security-guardian
    instruction: "Review the implementation for security issues"
```

## Context Variables

### Input Variables

```yaml
context:
  # Required - must be provided
  target_file: ""
  
  # Optional - has default value
  depth: "standard"
  include_tests: true
```

### Access Variables

```yaml
instruction: |
  Review {{target_file}} with {{depth}} depth.
  {{#if include_tests}}Include test analysis.{{/if}}
```

### Step Results

Each step's output is available to later steps:

```yaml
steps:
  - id: analyze
    instruction: "Analyze the code"
    
  - id: report
    instruction: |
      Previous analysis: {{analyze.result}}
      Create a summary report.
```

## Approval Gates

### Require Human Approval

```yaml
steps:
  - id: plan
    instruction: "Create a migration plan"
    
  - id: confirm
    requires_approval: true
    instruction: "Review the plan before proceeding"
    
  - id: execute
    instruction: "Execute the migration: {{plan.result}}"
```

### Approval Commands

```bash
# Check pending approvals
amp recipes approvals

# Approve
amp recipes approve [session-id] [step-id]

# Deny
amp recipes deny [session-id] [step-id] --reason "Need changes"
```

## Advanced Features

### Loops (foreach)

```yaml
steps:
  - id: get-files
    instruction: "List all Python files in src/"
    parse_json: true
    
  - id: review-each
    foreach: "{{get-files.result.files}}"
    instruction: "Review {{item}} for issues"
```

### Parallel Execution

```yaml
steps:
  - id: parallel-reviews
    foreach: "{{files}}"
    parallel: 3  # Run 3 at a time
    instruction: "Review {{item}}"
```

### Conditional Steps

```yaml
steps:
  - id: check
    instruction: "Does {{file}} have tests?"
    parse_json: true
    
  - id: write-tests
    condition: "{{check.result.has_tests}} == false"
    instruction: "Write tests for {{file}}"
```

### Error Handling

```yaml
steps:
  - id: risky-step
    instruction: "Try something that might fail"
    on_error: continue  # Don't stop on failure
    
  - id: handle-error
    condition: "{{risky-step.status}} == 'error'"
    instruction: "Handle the error: {{risky-step.error}}"
```

## Complete Examples

### Code Review Recipe

```yaml
name: code-review
description: Comprehensive code review with multiple specialists

context:
  file_path: ""
  review_depth: "standard"

steps:
  - id: design-review
    agent: foundation:zen-architect
    instruction: |
      Review {{file_path}} for design issues:
      - Architecture patterns
      - Code organization
      - Complexity
      
      Depth: {{review_depth}}

  - id: security-review
    agent: foundation:security-guardian
    instruction: |
      Review {{file_path}} for security issues:
      - Input validation
      - Authentication/authorization
      - Data handling

  - id: test-review
    agent: foundation:test-coverage
    instruction: |
      Analyze test coverage for {{file_path}}:
      - Existing tests
      - Missing coverage
      - Test quality

  - id: final-report
    instruction: |
      Create a code review report combining:
      
      ## Design Review
      {{design-review.result}}
      
      ## Security Review
      {{security-review.result}}
      
      ## Test Coverage
      {{test-review.result}}
      
      Include:
      - Summary of findings
      - Priority-ordered action items
      - Overall assessment
```

### Deployment Recipe with Approval

```yaml
name: deploy
description: Deploy to staging with approval gate

context:
  environment: "staging"
  version: ""

steps:
  - id: build
    instruction: "Build version {{version}}"
    
  - id: test
    instruction: "Run full test suite"
    
  - id: pre-deploy-check
    instruction: |
      Verify deployment readiness:
      - Build: {{build.result}}
      - Tests: {{test.result}}
      
      List any blockers.

  - id: approve-deploy
    requires_approval: true
    instruction: |
      Ready to deploy {{version}} to {{environment}}.
      
      Pre-check results:
      {{pre-deploy-check.result}}

  - id: deploy
    instruction: "Deploy {{version}} to {{environment}}"

  - id: verify
    instruction: "Verify deployment health"
```

## Recipe Validation

Check your recipe before running:

```bash
amp recipes validate my-recipe.yaml
```

Common validation errors:

| Error | Solution |
|-------|----------|
| Invalid YAML syntax | Check indentation, quotes |
| Unknown agent | Verify agent name |
| Missing context var | Add to context section |
| Circular reference | Check step dependencies |

## Recipe Author Agent

Get help creating recipes:

```
> Help me create a recipe for database migration
```

The `recipe-author` agent will:
- Ask clarifying questions
- Suggest best structure
- Generate valid YAML
- Add error handling

## Try It Yourself

### Exercise 1: Simple Recipe

```yaml
# hello-recipe.yaml
name: hello
description: Simple greeting recipe

context:
  name: ""

steps:
  - id: greet
    instruction: "Say hello to {{name}}"
```

```bash
amp recipes execute hello-recipe.yaml --context '{"name": "World"}'
```

### Exercise 2: Multi-Step Recipe

Create a recipe that:
1. Reads a file
2. Explains it
3. Suggests improvements

### Exercise 3: Add Approval Gate

Add an approval step before making changes.

## Best Practices

1. **Clear step IDs** - Use descriptive names like `analyze-deps` not `step1`
2. **One responsibility per step** - Keep steps focused
3. **Use appropriate agents** - Match agent specialty to task
4. **Add approval gates** - For destructive or important operations
5. **Handle errors** - Use `on_error` for steps that might fail
6. **Document context** - Make required inputs clear
