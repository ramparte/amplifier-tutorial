---
id: recipes-bundle
type: bundles
title: "Recipes Bundle"
---

# Recipes Bundle

The Recipes Bundle brings declarative workflow orchestration to Amplifier. Define multi-step AI agent workflows in YAML, execute them with automatic checkpointing, and add human-in-the-loop approval gates when needed.

## Overview

Recipes are declarative YAML specifications that define multi-step agent workflows. Instead of manually orchestrating complex tasks through multiple prompts, you define the workflow once and let the recipe engine handle execution, state management, and error recovery.

**Key Characteristics:**

- **Declarative**: Define *what* you want, not *how* to do it
- **Resumable**: Automatic checkpointing means interrupted workflows can continue
- **Composable**: Build complex workflows from simpler steps with dependencies
- **Auditable**: Full execution history for debugging and compliance
- **Approval Gates**: Pause workflows for human review at critical points

Recipes solve the coordination problem: orchestrating multiple AI agents across complex, multi-step tasks without manual intervention between each step.

## What's Included

### recipes Tool

The `recipes` tool is your interface to the recipe execution engine.

| Operation | Description |
|-----------|-------------|
| `execute` | Run a recipe from a YAML file with optional context |
| `resume` | Resume an interrupted session from checkpoint |
| `list` | List all active recipe sessions |
| `validate` | Validate recipe structure without executing |
| `approvals` | List pending approvals across all sessions |
| `approve` | Approve a stage to continue execution |
| `deny` | Deny a stage to halt execution |
| `cancel` | Cancel a running recipe session |

**Basic Usage:**

```yaml
# Execute a recipe
recipes:
  operation: execute
  recipe_path: workflows/code-review.yaml
  context:
    file_path: src/auth.py

# Resume interrupted session
recipes:
  operation: resume
  session_id: recipe_20251118_143022_a3f2
```

### recipe-author Agent

The `recipe-author` agent helps you create, validate, and refine recipe YAML specifications through conversation. It understands recipe schema, design patterns, and best practices.

**Use it to:**
- Create new recipes from requirements
- Validate existing recipe syntax
- Refine error handling and retry logic
- Add approval gates to workflows
- Optimize step dependencies

## Recipe Structure

### Flat Recipes

Flat recipes execute steps sequentially with explicit dependencies:

```yaml
name: code-review-workflow
description: Automated code review pipeline

context:
  file_path: ""  # Provided at runtime

steps:
  - id: analyze
    agent: foundation:zen-architect
    instruction: |
      Analyze the code structure in {{ file_path }}.
      Focus on architecture, patterns, and issues.
    
  - id: security
    agent: foundation:security-guardian
    instruction: Review {{ file_path }} for security vulnerabilities.
    depends_on: [analyze]
    
  - id: summarize
    agent: foundation:zen-architect
    instruction: |
      Create summary combining:
      - Architecture: {{ steps.analyze.output }}
      - Security: {{ steps.security.output }}
    depends_on: [analyze, security]
```

### Staged Recipes

Staged recipes group steps into approval stages. Execution pauses at stage boundaries for human approval:

```yaml
name: deploy-pipeline
description: Production deployment with approval gates

stages:
  - name: planning
    steps:
      - id: analyze-changes
        agent: foundation:zen-architect
        instruction: Analyze changes for deployment risks.
        
      - id: generate-plan
        agent: foundation:modular-builder
        instruction: Generate deployment plan.
        depends_on: [analyze-changes]

  - name: execution
    approval_required: true
    steps:
      - id: deploy
        agent: foundation:modular-builder
        instruction: Execute deployment according to plan.
```

### Step Configuration

Each step supports these options:

```yaml
steps:
  - id: unique-step-id           # Required: unique identifier
    agent: bundle:agent-name     # Required: agent to invoke
    instruction: |               # Required: task instructions
      What the agent should do.
      Can use {{ variable }} interpolation.
    
    # Optional configuration
    depends_on: [step-id]        # Steps that must complete first
    timeout: 300                 # Timeout in seconds (default: 300)
    retries: 3                   # Number of retry attempts
    on_error: continue           # continue | fail | skip
    condition: "{{ prev.success }}"  # Conditional execution
```

### Context and Variables

Recipes support context variables and step output interpolation:

```yaml
context:
  project_name: my-app
  environment: production
  
steps:
  - id: build
    agent: foundation:modular-builder
    instruction: Build {{ project_name }} for {{ environment }}.
    
  - id: deploy
    agent: foundation:modular-builder
    instruction: |
      Deploy using build artifacts.
      Build output: {{ steps.build.output }}
    depends_on: [build]
```

### Foreach Loops

Process collections with foreach:

```yaml
steps:
  - id: review-files
    foreach: "{{ files }}"
    as: current_file
    agent: foundation:zen-architect
    instruction: Review {{ current_file }} for code quality.
```

## When to Use

### Good Use Cases

✅ **Multi-agent workflows**: Tasks requiring multiple specialized agents working in sequence or parallel  
✅ **Approval-gated processes**: Workflows requiring human approval at critical points (deployments, data migrations)  
✅ **Repeatable processes**: Tasks performed regularly with the same structure  
✅ **Complex dependencies**: Steps with intricate dependencies and conditional execution  
✅ **Auditable workflows**: When you need full execution history for compliance  

### When Not to Use

❌ **Simple one-shot tasks**: Single agent can handle it? Invoke directly  
❌ **Highly dynamic workflows**: Next step depends entirely on unpredictable user input  
❌ **Exploratory work**: Don't know the steps upfront? Work interactively first  

## Try It Yourself

### Example 1: Create and Execute a Simple Recipe

Create `~/.amplifier/recipes/analyze-module.yaml`:

```yaml
name: analyze-module
description: Analyze a Python module for quality and security

context:
  module_path: ""

steps:
  - id: structure
    agent: foundation:explorer
    instruction: |
      Map the structure of {{ module_path }}.
      Identify key classes, functions, and dependencies.
      
  - id: quality
    agent: foundation:zen-architect
    instruction: |
      Review code quality in {{ module_path }}.
      Context: {{ steps.structure.output }}
    depends_on: [structure]
    
  - id: security
    agent: foundation:security-guardian
    instruction: |
      Check {{ module_path }} for security issues.
      Focus on input validation and data handling.
    depends_on: [structure]
    
  - id: report
    agent: foundation:zen-architect
    instruction: |
      Create summary report:
      - Quality: {{ steps.quality.output }}
      - Security: {{ steps.security.output }}
    depends_on: [quality, security]
```

Execute it:
```
Execute the analyze-module recipe for src/auth/
```

### Example 2: Staged Recipe with Approvals

Create `~/.amplifier/recipes/publish-content.yaml`:

```yaml
name: publish-content
description: Content publication with review gates

stages:
  - name: draft
    steps:
      - id: generate
        agent: foundation:modular-builder
        instruction: Generate content based on outline.
        
      - id: initial-review
        agent: foundation:zen-architect
        instruction: Review draft for clarity and structure.
        depends_on: [generate]

  - name: editorial
    approval_required: true
    steps:
      - id: fact-check
        agent: foundation:web-research
        instruction: Verify facts and claims in content.
        
      - id: style-edit
        agent: foundation:zen-architect
        instruction: Apply style guide and polish prose.

  - name: publish
    approval_required: true
    steps:
      - id: final-publish
        agent: foundation:modular-builder
        instruction: Publish to production.
```

### Example 3: Using the recipe-author Agent

```
Help me create a recipe for automated PR review that:
1. Analyzes code changes
2. Runs security checks
3. Validates test coverage
4. Requires approval before posting comments
```

The recipe-author agent will guide you through the design, asking clarifying questions and generating valid YAML.

### Example 4: Managing Recipe Sessions

```
# List active sessions
List my active recipe sessions

# Check pending approvals
Show pending recipe approvals

# Approve a stage
Approve the validation stage for session recipe_20251118_143022

# Resume interrupted session
Resume recipe session recipe_20251118_143022
```

## Best Practices

1. **Start simple**: Begin with flat recipes, add stages when you need approval gates
2. **Use meaningful IDs**: Step IDs appear in logs and error messages
3. **Handle errors explicitly**: Set `on_error` behavior based on criticality
4. **Test with validate**: Always validate before executing new recipes
5. **Version your recipes**: Store recipes in version control alongside code
6. **Keep steps focused**: Each step should do one thing well
7. **Use context for configuration**: Parameterize via context, don't hardcode

## Troubleshooting

**Recipe won't execute**: Run `validate` first to check for syntax errors  
**Step times out**: Increase `timeout` value or break into smaller pieces  
**Can't resume session**: Use `list` to verify session ID exists and isn't completed  
**Approval not working**: Ensure `approval_required: true` is on the stage, not the step  

## Related Bundles

- **foundation**: Provides agents commonly used in recipe steps
- **amplifier**: Core agent orchestration that recipes build upon

## Summary

The Recipes Bundle transforms complex multi-agent workflows into maintainable, repeatable, auditable processes. Define your workflow once in YAML, execute it reliably, and add human oversight where needed through approval gates.
