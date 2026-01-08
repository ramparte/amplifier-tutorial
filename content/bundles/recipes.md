---
id: bundle-recipes
type: bundles
title: "Recipes Bundle"
---

# Recipes Bundle

Multi-step workflow orchestration for complex, repeatable tasks.

## Overview

The recipes bundle adds:

- **Recipe execution** - Run YAML workflows
- **Approval gates** - Pause for human review
- **Session management** - Resume interrupted work
- **Context flow** - Pass results between steps

## Installation

```bash
# Add the bundle
amplifier bundle add git+https://github.com/microsoft/amplifier-bundle-recipes@main

# Use it
amplifier bundle use recipes
```

## What's Included

### Tools

| Tool | Purpose |
|------|---------|
| `recipes` | Execute, resume, validate recipes |

### Agents

| Agent | Purpose |
|-------|---------|
| `recipe-author` | Help create and debug recipes |
| `result-validator` | Validate step outcomes |

## Quick Start

### Run a Recipe

```bash
amplifier recipes execute examples/code-review.yaml \
  --context '{"file_path": "src/auth.py"}'
```

### In a Session

```
> Run the code-review recipe on src/main.py
```

### List Sessions

```bash
amplifier recipes list
```

## Recipe Operations

| Operation | Description |
|-----------|-------------|
| `execute` | Run a recipe |
| `resume` | Continue interrupted recipe |
| `validate` | Check recipe syntax |
| `list` | Show active sessions |
| `approvals` | Show pending approvals |
| `approve` | Approve a pending stage |
| `deny` | Deny a pending stage |

## Example Recipes

### Code Review

```yaml
name: code-review
description: Comprehensive code review

context:
  file_path: ""

steps:
  - id: design
    agent: foundation:zen-architect
    instruction: "Review {{file_path}} for design issues"

  - id: security
    agent: foundation:security-guardian
    instruction: "Check {{file_path}} for vulnerabilities"

  - id: report
    instruction: |
      Create review report:
      Design: {{design.result}}
      Security: {{security.result}}
```

### Dependency Update

```yaml
name: update-deps
description: Safe dependency updates

steps:
  - id: analyze
    instruction: "List outdated dependencies"

  - id: plan
    agent: foundation:zen-architect
    instruction: "Create update plan from {{analyze.result}}"

  - id: approve
    requires_approval: true
    instruction: "Review update plan"

  - id: execute
    instruction: "Execute updates from {{plan.result}}"

  - id: test
    instruction: "Run tests to verify"
```

### Security Audit

```yaml
name: security-audit
description: Full security review

context:
  target_dir: "src/"

steps:
  - id: deps
    instruction: "Check dependencies for vulnerabilities"

  - id: code
    agent: foundation:security-guardian
    instruction: "Review code in {{target_dir}}"

  - id: secrets
    instruction: "Scan for hardcoded secrets"

  - id: report
    instruction: |
      Create security report:
      - Dependencies: {{deps.result}}
      - Code review: {{code.result}}
      - Secrets: {{secrets.result}}
```

## Approval Workflows

### Define Approval Gate

```yaml
steps:
  - id: dangerous-step
    requires_approval: true
    instruction: "Execute production deployment"
```

### Respond to Approvals

```bash
# Check pending
amplifier recipes approvals

# Approve
amplifier recipes approve [session-id] dangerous-step

# Deny
amplifier recipes deny [session-id] dangerous-step \
  --reason "Need more testing"
```

## Context Variables

### Input Variables

```yaml
context:
  file_path: ""        # Required (no default)
  depth: "standard"    # Optional (has default)
```

### Step Results

```yaml
steps:
  - id: first
    instruction: "Analyze the code"

  - id: second
    instruction: |
      Based on analysis:
      {{first.result}}
      
      Suggest improvements.
```

## Advanced Features

### Foreach Loops

```yaml
steps:
  - id: get-files
    instruction: "List Python files"
    parse_json: true

  - id: review-each
    foreach: "{{get-files.result.files}}"
    instruction: "Review {{item}}"
```

### Parallel Execution

```yaml
steps:
  - id: parallel-reviews
    foreach: "{{files}}"
    parallel: 3  # 3 concurrent
    instruction: "Review {{item}}"
```

### Conditional Steps

```yaml
steps:
  - id: check
    instruction: "Are there tests?"
    parse_json: true

  - id: write-tests
    condition: "{{check.result.has_tests}} == false"
    instruction: "Write tests"
```

## Recipe Author Agent

Get help creating recipes:

```
> Help me create a recipe for deploying to staging
```

The `recipe-author` agent will:
- Ask clarifying questions
- Generate valid YAML
- Include best practices
- Add error handling

## Try It Yourself

### Exercise 1: Run Example Recipe

```bash
amplifier recipes execute examples/code-review.yaml \
  --context '{"file_path": "README.md"}'
```

### Exercise 2: Create Simple Recipe

```yaml
# my-recipe.yaml
name: explain
steps:
  - id: explain
    instruction: "Explain {{file}} in plain language"
```

```bash
amplifier recipes execute my-recipe.yaml \
  --context '{"file": "setup.py"}'
```

### Exercise 3: Use Approval Gate

```yaml
name: careful-work
steps:
  - id: plan
    instruction: "Create plan"
  - id: execute
    requires_approval: true
    instruction: "Execute plan"
```

## Source

```
github.com/microsoft/amplifier-bundle-recipes
├── bundle.yaml
├── agents/
│   ├── recipe-author.yaml
│   └── result-validator.yaml
├── examples/
│   ├── code-review.yaml
│   └── ...
└── docs/
    ├── RECIPE_SCHEMA.md
    └── BEST_PRACTICES.md
```
