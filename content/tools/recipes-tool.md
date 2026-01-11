---
id: tool-recipes
type: tools
title: "Recipes Tool"
---

# Recipes Tool

The Recipes Tool enables declarative workflow automation through YAML specifications.
Instead of manually orchestrating multi-step tasks, you define what needs to happen
and let Amplifier handle the execution, checkpointing, and error recovery.

## What are Recipes?

Recipes are declarative YAML workflows that define multi-step agent tasks. Think of
them as executable specifications that describe:

- **Steps**: Individual units of work performed by agents
- **Context**: Data that flows between steps
- **Dependencies**: How steps relate to each other
- **Checkpoints**: Automatic save points for resumability
- **Gates**: Human approval points for sensitive operations

A simple recipe might look like this:

```yaml
name: code-review
description: Review code changes and suggest improvements

steps:
  - name: analyze
    agent: foundation:zen-architect
    instruction: |
      Analyze the code in {{ file_path }} for:
      - Code quality issues
      - Potential bugs
      - Performance concerns
      
  - name: suggest
    agent: foundation:modular-builder
    instruction: |
      Based on the analysis, suggest specific improvements.
      Previous analysis: {{ steps.analyze.result }}
```

## Executing Recipes

To run a recipe, use the `execute` operation with the path to your YAML file:

```bash
amplifier run "execute recipe.yaml"
```

Or through the recipes tool directly:

```
recipes(operation="execute", recipe_path="my-recipe.yaml")
```

During execution, Amplifier:

1. Validates the recipe structure
2. Creates a new session for tracking
3. Executes steps sequentially
4. Persists state after each step
5. Returns the final result

## Passing Context

Recipes become powerful when you parameterize them with context variables.
Pass context as key-value pairs:

```bash
amplifier run "execute code-review.yaml with file_path=src/auth.py"
```

Or programmatically:

```
recipes(
  operation="execute",
  recipe_path="code-review.yaml",
  context={"file_path": "src/auth.py", "strict_mode": true}
)
```

Context variables are available in step instructions using Jinja2 templating:

```yaml
steps:
  - name: review
    instruction: |
      Review {{ file_path }} with strict_mode={{ strict_mode }}
```

### Context Accumulation

Each step's result is automatically added to context for subsequent steps:

```yaml
steps:
  - name: step_one
    instruction: "Generate a summary"
    # Result stored in steps.step_one.result
    
  - name: step_two
    instruction: |
      Expand on this summary: {{ steps.step_one.result }}
```

## Operations

The Recipes Tool supports several operations for managing workflow execution:

| Operation   | Purpose                                    | Required Parameters          |
|-------------|--------------------------------------------|------------------------------|
| `execute`   | Run a recipe from YAML file                | `recipe_path`                |
| `resume`    | Continue an interrupted session            | `session_id`                 |
| `list`      | List all active recipe sessions            | None                         |
| `validate`  | Check recipe YAML structure                | `recipe_path`                |
| `approvals` | List pending approvals across sessions     | None                         |
| `approve`   | Approve a stage to continue execution      | `session_id`, `stage_name`   |
| `deny`      | Deny a stage and stop execution            | `session_id`, `stage_name`   |

### Operation Examples

**Validate before running:**
```
recipes(operation="validate", recipe_path="deployment.yaml")
```

**Resume an interrupted session:**
```
recipes(operation="resume", session_id="recipe_20260110_143022_a3f2")
```

**Check pending approvals:**
```
recipes(operation="approvals")
```

## Approval Gates

For sensitive operations, recipes support staged execution with human approval gates.
This creates a human-in-the-loop workflow where execution pauses until approved.

### Defining Staged Recipes

```yaml
name: production-deploy
description: Deploy to production with approval gates

stages:
  - name: planning
    steps:
      - name: plan
        agent: foundation:zen-architect
        instruction: Create deployment plan for {{ service }}
    requires_approval: true
    
  - name: execution
    steps:
      - name: deploy
        agent: foundation:modular-builder
        instruction: |
          Execute deployment plan: {{ stages.planning.steps.plan.result }}
```

### Approval Workflow

1. Recipe executes until it hits an approval gate
2. Execution pauses and awaits human review
3. Human reviews the stage output
4. Human approves or denies:

```
# Approve to continue
recipes(operation="approve", session_id="...", stage_name="planning")

# Deny to stop execution
recipes(operation="deny", session_id="...", stage_name="planning", reason="Needs revision")
```

5. If approved, execution continues to the next stage

## Error Handling

Recipes include built-in error handling and retry logic:

```yaml
steps:
  - name: api_call
    agent: foundation:integration-specialist
    instruction: Call external API
    retry:
      max_attempts: 3
      delay_seconds: 5
    on_error: continue  # or 'fail' to stop execution
```

### Error Strategies

| Strategy   | Behavior                                      |
|------------|-----------------------------------------------|
| `fail`     | Stop execution immediately (default)          |
| `continue` | Log error and proceed to next step            |
| `retry`    | Attempt step again with configured backoff    |

## Best Practices

### 1. Keep Steps Focused

Each step should do one thing well:

```yaml
# Good: Single responsibility
steps:
  - name: analyze
    instruction: Analyze code structure
  - name: review
    instruction: Review for issues
  - name: suggest
    instruction: Suggest improvements

# Avoid: Monolithic steps
steps:
  - name: do_everything
    instruction: Analyze, review, and suggest all at once
```

### 2. Use Descriptive Names

Step and stage names become part of the context path:

```yaml
# Good: Clear, descriptive
steps:
  - name: security_audit
  - name: performance_review

# Avoid: Generic names
steps:
  - name: step1
  - name: step2
```

### 3. Validate Before Execution

Always validate recipes before running in production:

```
recipes(operation="validate", recipe_path="critical-workflow.yaml")
```

### 4. Design for Resumability

Structure recipes so they can resume gracefully:

- Make steps idempotent where possible
- Use clear checkpoints between major operations
- Store intermediate results in context

### 5. Use Approval Gates for Sensitive Operations

Add human checkpoints for:

- Production deployments
- Data migrations
- External API calls with side effects
- Operations that cannot be easily reversed

### 6. Document Your Recipes

Include clear descriptions:

```yaml
name: data-migration
description: |
  Migrates user data from legacy system to new schema.
  
  Prerequisites:
  - Backup completed
  - Maintenance window scheduled
  
  Context required:
  - source_db: Legacy database connection
  - target_db: New database connection
```

## Try It Yourself

Create a simple recipe to experience the workflow:

**1. Create `hello-recipe.yaml`:**

```yaml
name: hello-world
description: A simple introduction to recipes

steps:
  - name: greet
    agent: foundation:file-ops
    instruction: |
      Create a file called hello.txt with the message:
      "Hello from Amplifier Recipes!"
      
  - name: verify
    agent: foundation:file-ops
    instruction: |
      Read hello.txt and confirm it contains the greeting.
      Report the contents.
```

**2. Validate the recipe:**

```bash
amplifier run "validate hello-recipe.yaml"
```

**3. Execute it:**

```bash
amplifier run "execute hello-recipe.yaml"
```

**4. Check the results:**

The recipe will create `hello.txt` and verify its contents, demonstrating
the multi-step workflow with context passing between agents.

## Next Steps

- Learn about [Recipe Schema](/reference/recipe-schema) for advanced options
- Explore [Staged Recipes](/guides/staged-recipes) for complex workflows
- See [Recipe Examples](/examples/recipes) for real-world patterns
