---
id: tool-recipes
type: tools
title: "Recipes Tool"
---

# Recipes Tool

Execute multi-step workflows defined in YAML.

## Overview

The recipes tool runs declarative workflows:

| Operation | Purpose |
|-----------|---------|
| `execute` | Run a recipe |
| `resume` | Continue interrupted recipe |
| `list` | Show active sessions |
| `validate` | Check recipe syntax |
| `approvals` | Show pending approvals |
| `approve` / `deny` | Respond to approval gates |

## Requirements

```bash
# Add the recipes bundle
amplifier bundle add git+https://github.com/microsoft/amplifier-bundle-recipes@main
amplifier bundle use recipes
```

## Running Recipes

### Basic Execution

```
> Run the code-review recipe on src/auth.py
```

Or via CLI:

```bash
amplifier recipes execute recipes/code-review.yaml \
  --context '{"file_path": "src/auth.py"}'
```

### With Context Variables

Recipes accept input variables:

```yaml
# recipe.yaml
context:
  file_path: ""   # Required
  depth: "basic"  # Optional with default
```

```bash
amplifier recipes execute recipe.yaml \
  --context '{"file_path": "src/auth.py", "depth": "detailed"}'
```

## Managing Sessions

### List Active Sessions

```bash
amplifier recipes list
```

```
Active Recipe Sessions:
  ID                              Recipe          Status
  recipe_20260107_140000_abc123   code-review     running
  recipe_20260107_130000_def456   security-audit  awaiting_approval
```

### Resume Interrupted Session

```bash
amplifier recipes resume recipe_20260107_140000_abc123
```

Or in a session:

```
> Resume the code-review recipe
```

## Approval Gates

Some recipes pause for human approval:

```yaml
steps:
  - id: deploy
    requires_approval: true
    instruction: "Deploy to production"
```

### Check Pending Approvals

```bash
amplifier recipes approvals
```

```
Pending Approvals:
  Session: recipe_20260107_130000_def456
  Recipe: deploy-workflow
  Stage: deploy
  Waiting since: 10 minutes ago
```

### Approve a Stage

```bash
amplifier recipes approve recipe_20260107_130000_def456 deploy
```

### Deny a Stage

```bash
amplifier recipes deny recipe_20260107_130000_def456 deploy \
  --reason "Need more testing first"
```

## Validating Recipes

Check syntax before running:

```bash
amplifier recipes validate my-recipe.yaml
```

```
Recipe 'my-recipe' is valid.

Steps: 4
Agents used: zen-architect, modular-builder
Context variables: file_path (required), depth (optional)
```

## Example Session

```bash
# Start a recipe
$ amplifier recipes execute recipes/upgrade-deps.yaml \
    --context '{"project": "."}'

Starting recipe: upgrade-deps
Session ID: recipe_20260107_150000_xyz789

Step 1/4: analyze-deps
  [explorer] Analyzing dependencies...
  ✓ Found 15 outdated packages

Step 2/4: create-plan
  [zen-architect] Creating upgrade plan...
  ✓ Plan created

Step 3/4: approve-plan [REQUIRES APPROVAL]
  Review the upgrade plan before proceeding.
  
  To approve: amplifier recipes approve recipe_20260107_150000_xyz789 approve-plan
  To deny: amplifier recipes deny recipe_20260107_150000_xyz789 approve-plan

# In another terminal, approve it
$ amplifier recipes approve recipe_20260107_150000_xyz789 approve-plan

# Recipe continues...
Step 4/4: execute-upgrades
  [modular-builder] Executing upgrades...
  ✓ Dependencies upgraded

Recipe completed successfully!
```

## Try It Yourself

### Exercise 1: Run an Example

```bash
# See available example recipes
ls ~/.amplifier/bundles/recipes/examples/

# Run one
amplifier recipes execute examples/code-review.yaml \
  --context '{"file_path": "any-file.py"}'
```

### Exercise 2: Check Status

```bash
# List sessions
amplifier recipes list

# Check for pending approvals
amplifier recipes approvals
```

### Exercise 3: Validate Your Recipe

Create a simple recipe and validate it:

```yaml
# test-recipe.yaml
name: test
steps:
  - id: hello
    instruction: "Say hello"
```

```bash
amplifier recipes validate test-recipe.yaml
```

## Common Errors

### "Recipe not found"

Check the path:
```bash
ls recipes/  # Verify file exists
amplifier recipes execute ./recipes/my-recipe.yaml  # Use relative path
```

### "Context variable required"

Provide required context:
```bash
amplifier recipes execute recipe.yaml \
  --context '{"required_var": "value"}'
```

### "Invalid YAML"

Validate first:
```bash
amplifier recipes validate recipe.yaml
```

### "Session not found"

List available sessions:
```bash
amplifier recipes list
```
