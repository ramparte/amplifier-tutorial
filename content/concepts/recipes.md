---
id: recipes
type: concepts
title: "Understanding Recipes"
---

# Understanding Recipes

Recipes are declarative YAML specifications that define multi-step agent workflows. They let you orchestrate complex tasks that require multiple sequential or parallel operations, with built-in support for state persistence, checkpointing, and human approval gates.

## What is a Recipe?

A recipe is a blueprint for AI-assisted work. Rather than manually guiding an agent through each step of a complex process, you define the entire workflow upfront in a YAML file. The recipe system executes each step in order, passing context between steps and handling failures gracefully.

- **Declarative**: You specify *what* should happen, not *how*. The executor handles sequencing, context management, and error recovery.
- **Stateful**: Each step's output is stored in context and available to every subsequent step.
- **Resumable**: If a recipe is interrupted, it resumes from the last successful checkpoint. No work is lost.
- **Composable**: Recipes can invoke other recipes as sub-workflows, enabling modular, reusable components.

## How It Works

A recipe file has a straightforward structure: metadata at the top, optional context variables, and an ordered list of steps.

```yaml
name: "code-review"
description: "Analyze code for security and quality issues"
version: "1.0.0"

context:
  file_path: "src/auth.py"

steps:
  - id: "analyze"
    agent: "foundation:explorer"
    prompt: "Analyze the structure and patterns in {{file_path}}"
    output: "analysis"

  - id: "security-check"
    agent: "foundation:security-guardian"
    prompt: |
      Review {{file_path}} for security vulnerabilities.
      Previous analysis: {{analysis}}
    output: "security_findings"

  - id: "report"
    agent: "foundation:zen-architect"
    prompt: |
      Generate a review report combining:
      - Structure analysis: {{analysis}}
      - Security findings: {{security_findings}}
    output: "final_report"
```

### Context Flow Between Steps

The `{{variable}}` syntax is how data moves through a recipe. When a step declares `output: "analysis"`, its result becomes available to every later step as `{{analysis}}`. Context comes from step outputs, the top-level `context` block, and built-in variables like `{{recipe.name}}` and `{{session.id}}`. If a referenced variable doesn't exist at runtime, the recipe fails immediately with a clear error.

## Using Recipes

### Running a Recipe

To execute a recipe, ask Amplifier to run it. Recipes can live in bundles or as local files:

> Run the code review recipe on src/auth.py

Behind the scenes, the recipe tool supports these operations:

| Operation | Purpose |
|-----------|---------|
| `execute` | Run a recipe from a YAML file |
| `resume` | Resume an interrupted session |
| `list` | List active recipe sessions |
| `validate` | Check a recipe for errors without running it |
| `cancel` | Cancel a running session |

Recipes are referenced by path. Bundle recipes use the `@bundle:path` format (e.g., `@recipes:examples/code-review.yaml`), while local recipes use filesystem paths.

### Step Types

Not every step needs an LLM. Recipes support three step types:

**Agent steps** (default) spawn an LLM agent with a prompt:

```yaml
- id: "analyze"
  agent: "foundation:zen-architect"
  prompt: "Analyze the architecture of {{file_path}}"
  output: "analysis"
```

**Bash steps** execute shell commands directly, with no LLM overhead:

```yaml
- id: "run-tests"
  type: "bash"
  command: "npm test"
  output: "test_output"
```

**Recipe steps** invoke another recipe as a sub-workflow:

```yaml
- id: "security-audit"
  type: "recipe"
  recipe: "audits/security-audit.yaml"
  context:
    target_file: "{{file_path}}"
  output: "audit_result"
```

Sub-recipes receive only the context you explicitly pass to them. This isolation prevents accidental data leakage and makes each sub-recipe predictable and testable on its own.

### Conditional Execution

Steps can be skipped based on runtime conditions:

```yaml
- id: "critical-fix"
  condition: "{{severity}} == 'critical'"
  agent: "foundation:zen-architect"
  prompt: "Fix critical issues in {{file_path}}"
```

Conditions support `==`, `!=`, `<`, `>`, `<=`, `>=`, along with `and`, `or`, `not`, and parentheses for grouping.

### Iteration with Foreach

Process collections by iterating over list variables:

```yaml
- id: "discover-files"
  agent: "foundation:explorer"
  prompt: "List all Python files in {{directory}}"
  output: "files"

- id: "analyze-each"
  foreach: "{{files}}"
  as: "current_file"
  agent: "foundation:zen-architect"
  prompt: "Analyze {{current_file}} for issues"
  collect: "all_analyses"
```

The `collect` field gathers all iteration results into a list. Without it, `output` holds only the last iteration's result. Add `parallel: true` to run iterations concurrently, or `parallel: 5` for bounded concurrency.

### While Loops

For convergence-based workflows where you don't know iteration count upfront, use `while_condition`:

```yaml
- id: "refine"
  agent: "foundation:zen-architect"
  prompt: "Refine the design. Current state: {{draft}}"
  output: "refined"
  while_condition: "{{quality_score}} < 8"
  max_while_iterations: 10
  update_context:
    draft: "{{refined}}"
    quality_score: "{{refined.score}}"
  break_when: "{{quality_score}} >= 8"
```

The loop evaluates `while_condition` before each iteration, executes the step, applies `update_context` to mutate state, then checks `break_when` for early exit.

## Staged Recipes and Approval Gates

For high-stakes workflows that need human oversight, recipes support **staged mode** with approval gates. Instead of a flat `steps` list, you organize work into `stages`, each with an optional approval checkpoint.

```yaml
name: "production-deploy"
description: "Deploy with human approval gates"
version: "1.0.0"

stages:
  - name: "validation"
    steps:
      - id: "check-config"
        agent: "foundation:explorer"
        prompt: "Validate deployment configuration"
        output: "validation_result"
    approval:
      required: true
      prompt: "Validation complete: {{validation_result}}. Proceed to deploy?"
      timeout: 3600
      default: "deny"

  - name: "deployment"
    steps:
      - id: "deploy"
        type: "bash"
        command: "deploy.sh --env production"
        output: "deploy_result"
```

When execution reaches an approval gate, it pauses and returns a `paused_for_approval` status. Use `approvals` to list pending gates, `approve` or `deny` to respond (with an optional message or reason), then `resume` to continue. The approval message is available to subsequent steps as `{{_approval_message}}`, letting you pass human guidance into the workflow.

A recipe must use either `steps` (flat mode) or `stages` (staged mode), not both. Start with flat mode for simplicity and upgrade to staged mode when human oversight becomes necessary.

## Creating Your Own

Every recipe needs `name`, `description`, and `version` (semver format). Add `context` for input variables and `steps` for the workflow. Before running, validate your recipe to catch errors early:

> Validate my recipe at ./recipes/my-workflow.yaml

The validator checks structure, step ID uniqueness, variable references, and dependency ordering.

### Error Handling

Each step supports error handling configuration:

```yaml
- id: "optional-check"
  agent: "foundation:explorer"
  prompt: "Run optional validation"
  on_error: "continue"     # Don't fail the whole recipe
  timeout: 300             # 5 minute timeout
  retry:
    max_attempts: 3
    backoff: "exponential"
```

The `on_error` field accepts `"fail"` (default), `"continue"` (log and move on), or `"skip_remaining"` (stop without marking as failure).

### Where Recipes Live

| Location | Format | Use Case |
|----------|--------|----------|
| Bundle | `@recipes:path/name.yaml` | Shared, versioned workflows |
| Workspace | `./recipes/name.yaml` | Project-specific workflows |
| Absolute path | `/path/to/recipe.yaml` | One-off or testing |

## Best Practices

**Keep steps focused.** Each step should have a single, clear purpose. A step named "analyze-and-fix-and-report" is doing too much. Split it into three steps that pass context between them.

**Name outputs descriptively.** Use `output: "security_findings"` rather than `output: "result"`. Later steps read more naturally: `{{security_findings}}` vs `{{result}}`.

**Use bash steps for deterministic work.** Running tests, installing dependencies, or fetching data with `curl` don't need an LLM. Bash steps are faster and more predictable.

**Start flat, add stages when needed.** Begin with simple sequential steps. Add approval gates only when your workflow genuinely needs human checkpoints. Changing from flat to staged mode is a breaking change (major version bump).

**Use recipes for repeatable workflows, not ad-hoc tasks.** If you're exploring something interactively or running a one-shot task, direct agent conversation is simpler. Recipes shine when you'll run the same multi-step process repeatedly with different inputs.

**Test sub-recipes independently.** Because sub-recipes receive only explicitly passed context, each one should work on its own. This makes debugging and iteration much faster.

## Key Takeaways

1. **Recipes are declarative workflows**: Define steps in YAML, and the executor handles sequencing, state management, and error recovery.

2. **Context flows automatically**: Each step's output becomes a variable available to all subsequent steps via `{{variable_name}}` syntax.

3. **Three step types cover all needs**: Agent steps for reasoning, bash steps for deterministic commands, recipe steps for composing sub-workflows.

4. **Resumability is built-in**: Automatic checkpointing means interrupted recipes resume from the last successful step, not from scratch.

5. **Approval gates enable human oversight**: Staged recipes pause at approval gates, letting humans review progress before critical operations proceed.

6. **Iteration handles collections**: Foreach loops process lists with optional parallelism; while loops handle convergence-based workflows.

7. **Recipes vs direct conversation**: Use recipes for repeatable multi-step processes. Use direct agent interaction for exploratory or one-shot work.

---

## Related Concepts

- [Agents](./agents.md) - The agents that execute recipe steps
- [Bundles](./bundles.md) - How recipes are packaged and distributed
- [Modules](./modules.md) - The module system that recipes build upon
