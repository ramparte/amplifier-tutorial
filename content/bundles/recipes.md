---
id: recipes-bundle
type: bundles
title: "Recipes Bundle"
---

# Recipes Bundle

The Recipes Bundle provides workflow orchestration capabilities for Amplifier, enabling you to define multi-step AI agent workflows as declarative YAML specifications. Think of recipes as "playbooks" that coordinate complex, multi-step tasks with built-in state management, error handling, and human-in-the-loop approval gates.

## Overview

Recipes solve a fundamental challenge in AI-assisted development: coordinating complex tasks that require multiple steps, different agent capabilities, and human oversight. Instead of manually orchestrating each step, you define the workflow once and let the recipe system handle execution, state persistence, and error recovery.

Key characteristics:

- **Declarative**: Define what should happen, not how to execute it
- **Resumable**: Sessions persist state so interrupted workflows can continue
- **Composable**: Build complex workflows from simple, reusable steps
- **Observable**: Track progress and inspect state at any point
- **Human-in-the-loop**: Add approval gates for critical decisions

## What's Included

### The `recipes` Tool

The core tool for executing and managing recipe workflows:

```yaml
# Execute a recipe
recipes:
  operation: execute
  recipe_path: ./my-recipe.yaml
  context:
    target_file: src/auth.py

# Resume an interrupted session
recipes:
  operation: resume
  session_id: recipe_20251118_143022_a3f2

# List active sessions
recipes:
  operation: list

# Validate recipe structure
recipes:
  operation: validate
  recipe_path: ./my-recipe.yaml

# Approval operations (for staged recipes)
recipes:
  operation: approvals  # List pending approvals
  
recipes:
  operation: approve
  session_id: "..."
  stage_name: "planning"

recipes:
  operation: deny
  session_id: "..."
  stage_name: "planning"
  reason: "needs revision"
```

### The `recipe-author` Agent

A specialized agent for creating, validating, and refining recipe specifications:

- **Conversational design**: Guides you through recipe creation with clarifying questions
- **Schema validation**: Ensures recipes conform to the specification
- **Pattern guidance**: Recommends best practices for workflow orchestration
- **Error detection**: Identifies common mistakes before execution

## When to Use

### Good Use Cases

| Scenario | Why Recipes Help |
|----------|------------------|
| **Code review workflows** | Coordinate analysis, security check, and summary steps |
| **Multi-file refactoring** | Plan changes, implement across files, validate results |
| **Release processes** | Version bump, changelog, tests, PR creation with approvals |
| **Research tasks** | Gather information from multiple sources, synthesize findings |
| **Migration projects** | Analyze scope, plan phases, execute with checkpoints |

### When Not to Use

- **Simple, single-step tasks**: Direct agent invocation is simpler
- **Highly interactive work**: Recipes are for autonomous execution
- **Unpredictable workflows**: If you can't define steps upfront, use agents directly

## Recipe Anatomy

### Basic Structure (Flat Recipe)

```yaml
name: code-review
description: Review code for quality and security issues

# Input parameters
inputs:
  file_path:
    type: string
    required: true
    description: Path to the file to review

# Sequential steps
steps:
  - id: analyze
    agent: foundation:zen-architect
    instruction: |
      Analyze the code structure in {{ file_path }}.
      Identify complexity issues and improvement opportunities.
    
  - id: security
    agent: foundation:security-guardian
    instruction: |
      Review {{ file_path }} for security vulnerabilities.
      Check for OWASP Top 10 issues.
    
  - id: summarize
    agent: foundation:file-ops
    instruction: |
      Create a review summary combining:
      - Analysis findings: {{ steps.analyze.result }}
      - Security findings: {{ steps.security.result }}
```

### Staged Recipe (With Approval Gates)

```yaml
name: release-process
description: Coordinated release with human approval gates

inputs:
  version:
    type: string
    required: true

stages:
  planning:
    approval_required: true
    steps:
      - id: changelog
        agent: foundation:explorer
        instruction: |
          Gather all changes since last release.
          Draft changelog for version {{ version }}.
          
  implementation:
    approval_required: true
    steps:
      - id: version_bump
        agent: foundation:file-ops
        instruction: |
          Update version to {{ version }} in package files.
          
      - id: create_pr
        agent: foundation:git-ops
        instruction: |
          Create release PR with changelog from planning stage.
```

## Key Concepts

### Context and Variable Interpolation

Recipes support Jinja2-style templating for dynamic values:

```yaml
# Input variables
instruction: "Review {{ file_path }}"

# Step results
instruction: "Build on analysis: {{ steps.analyze.result }}"

# Stage results (staged recipes)
instruction: "Use plan from: {{ stages.planning.result }}"
```

### Error Handling

Configure behavior when steps fail:

```yaml
steps:
  - id: risky_operation
    agent: foundation:modular-builder
    instruction: "Attempt complex refactoring"
    on_error: continue  # Options: fail (default), continue, retry
    retry:
      max_attempts: 3
      delay: 5  # seconds
```

### Timeouts

Prevent runaway executions:

```yaml
steps:
  - id: long_running
    agent: foundation:explorer
    instruction: "Deep codebase analysis"
    timeout: 300  # 5 minutes
```

### Foreach Loops

Iterate over collections:

```yaml
steps:
  - id: review_files
    foreach: "{{ input_files }}"
    as: current_file
    agent: foundation:zen-architect
    instruction: "Review {{ current_file }}"
```

### Conditional Execution

Run steps based on conditions:

```yaml
steps:
  - id: deploy
    when: "{{ steps.tests.result.success == true }}"
    agent: foundation:integration-specialist
    instruction: "Deploy to staging"
```

## Working with Sessions

### Session Lifecycle

1. **Created**: Recipe parsed and validated
2. **Running**: Steps executing sequentially
3. **Paused**: Waiting for approval (staged recipes)
4. **Completed**: All steps finished successfully
5. **Failed**: Unrecoverable error encountered

### Listing and Resuming

```bash
# In Amplifier, use the recipes tool:

# List all active sessions
recipes operation=list

# Resume a specific session
recipes operation=resume session_id=recipe_20251118_143022_a3f2
```

### Approval Workflow

For staged recipes with `approval_required: true`:

```bash
# Check pending approvals
recipes operation=approvals

# Approve a stage to continue
recipes operation=approve session_id="..." stage_name="planning"

# Deny with reason (stops execution)
recipes operation=deny session_id="..." stage_name="planning" reason="needs changes"
```

## Try It Yourself

### Example 1: Simple Code Review Recipe

Create `code-review.yaml`:

```yaml
name: code-review
description: Comprehensive code review workflow

inputs:
  file_path:
    type: string
    required: true

steps:
  - id: structure
    agent: foundation:zen-architect
    instruction: |
      Review the code structure of {{ file_path }}.
      Assess: complexity, readability, adherence to SOLID principles.
      
  - id: security
    agent: foundation:security-guardian
    instruction: |
      Security review of {{ file_path }}.
      Check for common vulnerabilities.
      
  - id: report
    agent: foundation:file-ops
    instruction: |
      Generate a markdown review report combining:
      
      ## Structure Review
      {{ steps.structure.result }}
      
      ## Security Review
      {{ steps.security.result }}
```

Execute it:

```
Use the recipes tool to execute code-review.yaml with file_path set to src/auth.py
```

### Example 2: Research and Summarize

Create `research.yaml`:

```yaml
name: topic-research
description: Research a topic and produce a summary

inputs:
  topic:
    type: string
    required: true
  depth:
    type: string
    default: "moderate"

steps:
  - id: gather
    agent: foundation:web-research
    instruction: |
      Research "{{ topic }}" with {{ depth }} depth.
      Find authoritative sources and key information.
      
  - id: analyze
    agent: foundation:zen-architect
    instruction: |
      Analyze the research findings:
      {{ steps.gather.result }}
      
      Identify key themes, contradictions, and knowledge gaps.
      
  - id: synthesize
    agent: foundation:file-ops
    instruction: |
      Create a comprehensive summary document on "{{ topic }}":
      
      Research: {{ steps.gather.result }}
      Analysis: {{ steps.analyze.result }}
```

### Example 3: Using the Recipe Author

For complex recipes, use the recipe-author agent:

```
I need a recipe that:
1. Scans a codebase for TODO comments
2. Categorizes them by priority
3. Creates GitHub issues for high-priority items
4. Requires approval before creating issues
```

The recipe-author agent will guide you through the design, asking clarifying questions and generating valid YAML.

## Best Practices

1. **Start simple**: Begin with flat recipes before adding stages
2. **Use meaningful step IDs**: They appear in logs and result references
3. **Provide clear instructions**: Agents work best with specific, detailed prompts
4. **Add approval gates sparingly**: Only where human judgment is truly needed
5. **Test with validate first**: Catch schema errors before execution
6. **Keep steps focused**: One clear objective per step

## Common Patterns

### Sequential Analysis Pipeline

```yaml
steps:
  - id: gather    # Collect information
  - id: analyze   # Process findings
  - id: decide    # Make recommendations
  - id: implement # Execute decisions
```

### Fan-Out/Fan-In

```yaml
steps:
  - id: analyze_all
    foreach: "{{ files }}"
    as: file
    agent: foundation:explorer
    
  - id: combine
    instruction: "Synthesize: {{ steps.analyze_all.results }}"
```

### Approval Checkpoint

```yaml
stages:
  planning:
    approval_required: true
    steps: [...]
    
  execution:
    # Only runs after planning is approved
    steps: [...]
```

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Recipe won't validate | Check YAML syntax, ensure required fields present |
| Step fails immediately | Verify agent name is correct, check instruction format |
| Variables not interpolating | Use `{{ }}` syntax, check step ID references |
| Session won't resume | Verify session_id, check if session completed or failed |
| Approval stuck | Use `recipes operation=approvals` to see pending gates |
