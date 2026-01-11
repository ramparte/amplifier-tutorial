---
id: recipes
type: concepts
title: "Understanding Recipes"
---

# Understanding Recipes

Recipes are declarative YAML specifications that define multi-step agent workflows. They allow you to orchestrate complex tasks by breaking them into sequential steps, each potentially delegating to different agents. Recipes provide structure, reproducibility, and human oversight for AI-driven processes.

## What is a Recipe?

A Recipe is a blueprint for agent orchestration. Rather than manually invoking agents step-by-step, you define the entire workflow upfront in a YAML file. The recipe executor then handles:

- **Sequential execution** - Steps run in order, with each step's output available to subsequent steps
- **State persistence** - Context accumulates across steps, building a shared understanding
- **Automatic checkpointing** - Sessions can be resumed if interrupted
- **Error handling** - Built-in retry logic and failure management
- **Approval gates** - Human-in-the-loop checkpoints for critical decisions

Think of recipes as the difference between giving verbal instructions one at a time versus handing someone a complete written procedure. Both accomplish the goal, but the written procedure is reproducible, auditable, and can be improved over time.

### A Simple Example

```yaml
name: code-review
description: Review code changes and suggest improvements

steps:
  - id: analyze
    agent: foundation:explorer
    instruction: |
      Analyze the code in {{ file_path }}.
      Identify the main purpose, structure, and any patterns used.

  - id: review
    agent: foundation:zen-architect
    instruction: |
      Based on the analysis:
      {{ steps.analyze.result }}
      
      Review this code for:
      - Adherence to simplicity principles
      - Potential bugs or edge cases
      - Opportunities for improvement
```

This recipe defines two steps: first exploring the code structure, then reviewing it with architectural expertise. The second step receives the output of the first through template variables.

## When to Use Recipes

Recipes excel in scenarios where you need:

### Reproducible Workflows

When you find yourself repeatedly performing the same multi-agent task, a recipe captures that process. Instead of remembering which agents to invoke and in what order, you run the recipe and get consistent results.

**Examples:**
- Code review pipelines
- Documentation generation
- Release preparation checklists
- Security audits

### Complex Multi-Step Tasks

Some tasks naturally decompose into phases that build on each other. Recipes make these dependencies explicit and ensure proper sequencing.

**Examples:**
- Feature implementation (design → implement → test → document)
- Bug investigation (reproduce → analyze → fix → verify)
- Migration tasks (assess → plan → execute → validate)

### Tasks Requiring Human Oversight

When certain decisions shouldn't be fully automated, recipes support approval gates. Execution pauses until a human reviews and approves (or rejects) continuing.

**Examples:**
- Production deployments
- Security-sensitive changes
- Decisions with significant business impact

### Collaborative Workflows

Recipes create a shared understanding between team members. Anyone can run the recipe and understand exactly what steps will be taken, making handoffs and reviews straightforward.

## Recipe Structure

Every recipe follows a consistent structure with required and optional elements.

### Required Elements

```yaml
name: my-workflow          # Unique identifier for the recipe
description: What it does  # Human-readable description

steps:                     # List of steps to execute
  - id: step-one           # Unique step identifier
    agent: agent:name      # Agent to invoke
    instruction: |         # Task for the agent
      What to do...
```

### Optional Elements

```yaml
# Context variables (can be overridden at execution time)
context:
  default_branch: main
  environment: staging

# Global settings
settings:
  timeout: 300             # Default timeout per step (seconds)
  retry_count: 2           # Default retry attempts on failure

# Step-level options
steps:
  - id: example
    agent: foundation:explorer
    instruction: Explore the codebase
    timeout: 600           # Override default timeout
    retry_count: 3         # Override default retries
    on_error: continue     # continue, fail, or skip
```

### Flat vs. Staged Recipes

Recipes come in two flavors:

**Flat recipes** execute all steps sequentially without pause:

```yaml
name: simple-task
steps:
  - id: step-one
    agent: foundation:explorer
    instruction: First task
  - id: step-two
    agent: foundation:file-ops
    instruction: Second task
```

**Staged recipes** group steps into stages with approval gates between them:

```yaml
name: careful-deployment
stages:
  - name: planning
    steps:
      - id: analyze
        agent: foundation:zen-architect
        instruction: Plan the deployment
    requires_approval: true
    
  - name: execution
    steps:
      - id: deploy
        agent: foundation:modular-builder
        instruction: Execute the deployment plan
```

## Steps and Context

Steps are the fundamental units of a recipe. Each step invokes an agent with specific instructions and can access accumulated context from previous steps.

### Step Anatomy

```yaml
steps:
  - id: unique-identifier    # Required: referenced by other steps
    agent: bundle:agent-name # Required: which agent to use
    instruction: |           # Required: what to do
      Detailed instructions for the agent.
      Can span multiple lines.
    
    # Optional fields
    timeout: 300             # Seconds before timeout
    retry_count: 2           # Retries on failure
    on_error: fail           # fail, continue, or skip
    condition: "{{ steps.previous.success }}"  # Conditional execution
```

### Context Flow

Context flows through recipes in two ways:

**1. Explicit context variables** - Defined at recipe level or passed at execution:

```yaml
context:
  project_name: my-app
  target_branch: main

steps:
  - id: setup
    agent: foundation:file-ops
    instruction: |
      Working on project: {{ project_name }}
      Target branch: {{ target_branch }}
```

**2. Step results** - Output from previous steps:

```yaml
steps:
  - id: discover
    agent: foundation:explorer
    instruction: Find all Python files in src/

  - id: analyze
    agent: lsp-python:python-code-intel
    instruction: |
      Analyze these files:
      {{ steps.discover.result }}
```

### Template Syntax

Recipes use Jinja2-style templates for variable interpolation:

| Pattern | Description |
|---------|-------------|
| `{{ variable }}` | Insert a context variable |
| `{{ steps.id.result }}` | Insert a step's result |
| `{{ steps.id.success }}` | Boolean: did the step succeed? |
| `{% if condition %}...{% endif %}` | Conditional content |
| `{% for item in list %}...{% endfor %}` | Loop over items |

### Advanced Context Features

**Foreach loops** execute a step multiple times:

```yaml
steps:
  - id: review-each
    agent: foundation:zen-architect
    foreach: "{{ files }}"
    instruction: |
      Review the file: {{ item }}
```

**Conditional execution** skips steps based on conditions:

```yaml
steps:
  - id: deploy-prod
    agent: foundation:modular-builder
    condition: "{{ environment == 'production' }}"
    instruction: Deploy to production servers
```

## Approval Gates

Approval gates pause recipe execution until a human approves or denies continuation. They provide critical oversight for high-stakes workflows.

### Defining Approval Gates

In staged recipes, add `requires_approval: true` to a stage:

```yaml
stages:
  - name: analysis
    steps:
      - id: security-scan
        agent: foundation:security-guardian
        instruction: Scan for vulnerabilities
    requires_approval: true
    approval_message: |
      Security scan complete. Review findings before proceeding.
      
  - name: deployment
    steps:
      - id: deploy
        agent: foundation:modular-builder
        instruction: Deploy the application
```

### Managing Approvals

When a recipe reaches an approval gate, it pauses and waits. You can manage pending approvals:

```yaml
# List all pending approvals
recipes operation=approvals

# Approve a stage to continue
recipes operation=approve session_id=xxx stage_name=analysis

# Deny a stage (stops execution)
recipes operation=deny session_id=xxx stage_name=analysis reason="Found critical vulnerability"
```

### Approval Metadata

When approving, you can include metadata that becomes available to subsequent steps:

```yaml
# The approval context is available in later steps
steps:
  - id: post-approval
    agent: foundation:modular-builder
    instruction: |
      Approval notes: {{ approval.notes }}
      Approved by: {{ approval.user }}
```

### Best Practices for Approval Gates

1. **Include clear context** - The approval message should summarize what was done and what will happen next
2. **Use sparingly** - Too many gates slow workflows; reserve for genuinely critical checkpoints
3. **Provide actionable information** - Include links, summaries, or specific items to review
4. **Consider timeouts** - Long-pending approvals may indicate a stale workflow

## Recipe Operations

The recipe system supports several operations:

| Operation | Description |
|-----------|-------------|
| `execute` | Run a recipe from a YAML file |
| `resume` | Continue an interrupted session |
| `list` | Show active recipe sessions |
| `validate` | Check recipe structure without running |
| `approvals` | List pending approvals |
| `approve` | Approve a pending stage |
| `deny` | Deny a pending stage |

### Execution Example

```yaml
# Execute a recipe with context
recipes operation=execute recipe_path=review.yaml context={"file_path": "src/main.py"}

# Resume an interrupted session
recipes operation=resume session_id=recipe_20260110_143022_a3f2

# Validate before running
recipes operation=validate recipe_path=new-recipe.yaml
```

## Error Handling

Recipes provide flexible error handling at multiple levels:

### Step-Level Error Handling

```yaml
steps:
  - id: risky-operation
    agent: foundation:integration-specialist
    instruction: Call external API
    retry_count: 3           # Retry up to 3 times
    timeout: 60              # Timeout after 60 seconds
    on_error: continue       # Don't fail the whole recipe
```

### Error Strategies

| Strategy | Behavior |
|----------|----------|
| `fail` | Stop the recipe immediately (default) |
| `continue` | Log the error and proceed to next step |
| `skip` | Skip dependent steps and continue |

### Accessing Error Information

Failed steps provide error context:

```yaml
steps:
  - id: handle-failure
    condition: "{{ not steps.risky-operation.success }}"
    agent: foundation:bug-hunter
    instruction: |
      The previous step failed with:
      {{ steps.risky-operation.error }}
      
      Investigate and suggest fixes.
```

## Key Takeaways

1. **Recipes are declarative workflows** - Define what should happen, not how to orchestrate it manually

2. **Steps build on each other** - Each step can access results from previous steps through template variables

3. **Two flavors exist** - Flat recipes for simple sequences, staged recipes for workflows needing approval gates

4. **Context flows automatically** - Variables and results accumulate, giving later steps full awareness of earlier work

5. **Approval gates enable oversight** - Pause execution at critical points for human review and decision-making

6. **Error handling is flexible** - Configure retries, timeouts, and failure strategies per-step or globally

7. **Sessions are resumable** - Interrupted recipes can continue from where they left off

8. **Validation prevents surprises** - Check recipe structure before execution to catch errors early

Recipes transform ad-hoc agent orchestration into reproducible, auditable processes. Start simple with flat recipes, add stages and approvals as workflows mature, and leverage context flow to build sophisticated multi-agent pipelines.
