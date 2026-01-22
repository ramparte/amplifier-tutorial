---
id: recipes
type: concepts
title: "Understanding Recipes"
---

# Understanding Recipes

Recipes are declarative YAML specifications that define multi-step AI agent workflows. They provide a structured way to orchestrate complex tasks that require multiple sequential or parallel operations, with built-in support for state persistence, error handling, and human approval gates.

## What is a Recipe?

A recipe is essentially a blueprint for AI-assisted work. Rather than manually guiding an agent through each step of a complex process, you define the entire workflow upfront in a YAML file. The recipe system then executes each step in order, passing context between steps and handling failures gracefully.

Think of recipes like cooking recipes: they specify ingredients (context), steps (operations), and expected outcomes. Just as a cooking recipe ensures consistent results regardless of who follows it, an agent recipe ensures consistent, reproducible workflows.

### Core Characteristics

**Declarative**: You specify *what* should happen, not *how* to do it. The recipe executor handles the mechanics of step execution, context management, and error recovery.

**Sequential with State**: Each step can access the accumulated context from all previous steps. This creates a natural flow where early steps gather information that later steps act upon.

**Resumable**: If a recipe is interrupted (network failure, timeout, manual stop), it can be resumed from the last successful checkpoint. No work is lost.

**Composable**: Recipes can be simple (3-4 steps) or complex (dozens of steps with conditional branching). You build what you need.

## When to Use Recipes

Recipes shine in scenarios where manual orchestration would be tedious, error-prone, or inconsistent.

### Ideal Use Cases

**Code Review Workflows**: A recipe can analyze code changes, check for security issues, verify test coverage, and generate a comprehensive review report - all in one automated flow.

**Research and Synthesis**: Gather information from multiple sources, analyze patterns, synthesize findings, and produce a structured report.

**Multi-Stage Deployments**: Validate configuration, run tests, build artifacts, deploy to staging, run smoke tests, and optionally promote to production.

**Document Generation**: Analyze a codebase, extract documentation, generate API references, and compile everything into a cohesive document.

**Refactoring Operations**: Analyze current code, identify patterns to change, plan modifications, execute changes across files, and verify results.

### When NOT to Use Recipes

- **Simple one-shot tasks**: If a single agent call suffices, skip the recipe overhead
- **Highly interactive work**: Recipes are designed for autonomous execution, not back-and-forth conversation
- **Exploratory tasks**: When you don't know the steps upfront, manual guidance is better

## Recipe Structure

A recipe file has a clear structure with metadata, context, and steps.

### Basic Anatomy

```yaml
# Recipe metadata
name: code-review
description: Comprehensive code review workflow
version: "1.0"

# Input context (variables passed at execution time)
context:
  required:
    - file_path    # Path to file being reviewed
  optional:
    - focus_areas  # Specific areas to emphasize

# The workflow steps
steps:
  - name: analyze-structure
    agent: foundation:explorer
    prompt: |
      Analyze the structure of {{ file_path }}.
      Identify key components, dependencies, and patterns.

  - name: security-check
    agent: foundation:security-guardian
    prompt: |
      Review {{ file_path }} for security vulnerabilities.
      Previous analysis: {{ steps.analyze-structure.result }}

  - name: generate-report
    agent: foundation:file-ops
    prompt: |
      Generate a code review report based on:
      - Structure analysis: {{ steps.analyze-structure.result }}
      - Security findings: {{ steps.security-check.result }}
```

### Key Elements

**name**: A unique identifier for the recipe. Used for execution and logging.

**description**: Human-readable explanation of what the recipe does.

**context**: Defines inputs the recipe expects. Required context must be provided at execution time; optional context has defaults or can be omitted.

**steps**: The ordered list of operations to execute. Each step has a name, an agent to invoke, and a prompt template.

## Steps and Context

Steps are the building blocks of recipes. Each step represents a discrete unit of work performed by an agent.

### Step Definition

```yaml
steps:
  - name: step-identifier        # Unique name within recipe
    agent: foundation:explorer   # Agent to invoke
    prompt: |                    # Instructions for the agent
      Your task description here.
      You can reference context: {{ variable_name }}
      And previous results: {{ steps.previous-step.result }}
```

### Context Flow

Context flows through recipes in a predictable way:

1. **Initial Context**: Variables provided at recipe execution
2. **Step Results**: Each step's output is stored and accessible to later steps
3. **Accumulated State**: Later steps have access to all prior results

```yaml
# Step 1 receives initial context
- name: gather-info
  prompt: "Analyze {{ target_directory }}"  # target_directory from initial context

# Step 2 receives initial context + step 1 results
- name: process-info
  prompt: |
    Based on: {{ steps.gather-info.result }}
    Process the information...

# Step 3 receives everything
- name: synthesize
  prompt: |
    Original target: {{ target_directory }}
    Gathered info: {{ steps.gather-info.result }}
    Processed info: {{ steps.process-info.result }}
```

### Template Syntax

Recipes use Jinja2-style templating for dynamic content:

- `{{ variable }}`: Insert a context variable
- `{{ steps.step-name.result }}`: Insert a previous step's result
- `{{ context.required_var }}`: Explicitly reference required context

### Step Options

Steps support additional configuration:

```yaml
- name: critical-operation
  agent: foundation:modular-builder
  prompt: "Implement the feature..."
  
  # Error handling
  on_error: continue    # Options: fail (default), continue, retry
  max_retries: 3        # For retry strategy
  
  # Timeouts
  timeout: 300          # Seconds before timeout
  
  # Conditional execution
  when: "{{ steps.validation.result.passed }}"
```

## Approval Gates

For workflows that require human oversight, recipes support approval gates. These pause execution at critical points, allowing humans to review progress before continuing.

### Staged Recipes

Approval gates are implemented through staged recipes - recipes divided into stages that require explicit approval between them.

```yaml
name: production-deployment
description: Deploy to production with approval gates

stages:
  - name: preparation
    steps:
      - name: validate-config
        agent: foundation:explorer
        prompt: "Validate deployment configuration..."
      
      - name: run-tests
        agent: foundation:test-coverage
        prompt: "Execute full test suite..."

  - name: staging-deployment
    approval_required: true
    approval_message: |
      Preparation complete. Test results:
      {{ steps.run-tests.result }}
      
      Approve to deploy to staging?
    steps:
      - name: deploy-staging
        agent: foundation:modular-builder
        prompt: "Deploy to staging environment..."

  - name: production-release
    approval_required: true
    approval_message: |
      Staging deployment successful.
      Approve production release?
    steps:
      - name: deploy-production
        agent: foundation:modular-builder
        prompt: "Deploy to production..."
```

### Approval Workflow

1. Recipe executes until it hits a stage with `approval_required: true`
2. Execution pauses and the approval message is displayed
3. Human reviews the state and either approves or denies
4. On approval: execution continues to the next stage
5. On denial: execution stops with the provided reason

### Managing Approvals

```bash
# List pending approvals across all sessions
amp recipes approvals

# Approve a pending stage
amp recipes approve --session-id <id> --stage-name staging-deployment

# Deny with reason
amp recipes deny --session-id <id> --stage-name production-release \
  --reason "Test coverage below threshold"
```

### Use Cases for Approval Gates

- **Production deployments**: Human sign-off before going live
- **Data migrations**: Review migration plan before execution
- **Bulk operations**: Approve after seeing what will be affected
- **Compliance workflows**: Required human verification for audit trails

## Advanced Features

### Foreach Loops

Execute steps for each item in a collection:

```yaml
- name: process-files
  foreach: "{{ discovered_files }}"
  as: current_file
  steps:
    - name: analyze
      agent: foundation:explorer
      prompt: "Analyze {{ current_file }}..."
```

### Conditional Steps

Skip steps based on conditions:

```yaml
- name: security-scan
  when: "{{ context.security_required }}"
  agent: foundation:security-guardian
  prompt: "Perform security scan..."
```

### Error Recovery

Configure how failures are handled:

```yaml
- name: optional-optimization
  agent: foundation:zen-architect
  prompt: "Optimize if possible..."
  on_error: continue  # Don't fail the whole recipe
```

## Key Takeaways

1. **Recipes are declarative workflows**: Define what you want, not how to do it. The recipe system handles execution mechanics.

2. **State flows automatically**: Each step can access all previous results. No manual state management required.

3. **Resumability is built-in**: Interrupted recipes can be resumed. Checkpointing happens automatically.

4. **Approval gates enable human oversight**: For critical workflows, pause and require human approval before proceeding.

5. **Start simple, grow as needed**: Begin with linear sequences of steps. Add stages, conditions, and loops only when your workflow demands it.

6. **Recipes are reusable**: Once defined, a recipe can be executed repeatedly with different inputs. Build a library of workflows for common tasks.

7. **Think in discrete steps**: Each step should have a clear, focused purpose. Avoid monolithic steps that try to do everything.

Recipes transform complex, multi-step processes into repeatable, reliable workflows. They bridge the gap between manual agent interaction and fully automated pipelines, giving you control over the workflow design while letting AI handle the execution.
