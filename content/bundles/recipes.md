---
id: recipes-bundle
type: bundle
title: "Recipes Bundle"
---

# Recipes Bundle

## What is the Recipes Bundle?

You've used Amplifier to read files, run commands, and delegate to agents. Each of those is a single action. But real work often looks like a pipeline: analyze the code, then check for security issues, then generate a report, then wait for someone to approve before deploying. That's a multi-step workflow — and doing it manually means typing each prompt, waiting, copying context forward, and hoping nothing breaks in the middle.

The Recipes Bundle adds declarative workflow orchestration to Amplifier. You define multi-step agent workflows in YAML, and the recipe engine handles execution order, state management, checkpointing, and error recovery. If a recipe gets interrupted — network glitch, laptop closed, timeout — it resumes from the last completed step, not from scratch.

Recipes follow a simple philosophy: define *what* should happen, not *how*. You declare the steps, the agents, and the context flow. The engine does the rest.

## What's Included

The bundle provides three components that work together:

**The `recipes` tool** — your interface to the execution engine. It runs recipes, manages sessions, handles approvals, and validates YAML before you commit to running it. See the [Recipes Tool](../tools/recipes-tool.md) page for the full operation reference.

**The `recipe-author` agent** — a specialist that creates, validates, and refines recipe YAML through conversation. Describe your workflow in plain English, and it generates a valid specification, asking clarifying questions along the way.

**The `result-validator` agent** — an objective verifier that checks whether a recipe's output matches your original intent. Use it after execution to assess whether the outcome meets your goals.

The bundle also ships with example recipes and starter templates you can customize.

## Getting Started

The simplest way to use a recipe is to run one that already exists. The bundle includes several examples you can try immediately.

> Run the code review recipe on src/auth.py

```
[Tool: recipes]
  operation: execute
  recipe_path: "@recipes:examples/code-review-recipe.yaml"
  context: { "file_path": "src/auth.py" }

Session started: recipe_20260402_143022_a3f2
Step 1/3 [analyze]: Analyzing src/auth.py...
Step 2/3 [security-check]: Reviewing for vulnerabilities...
Step 3/3 [report]: Generating final report...
Recipe completed successfully.
```

Each step runs in order. The output from `analyze` flows into `security-check` as context, and both flow into `report`. You didn't have to copy anything between steps — the engine handles context accumulation automatically. Before running a new recipe, validate it first:

> Validate the recipe at ./recipes/my-workflow.yaml

```
[Tool: recipes]
  operation: validate
  recipe_path: "./recipes/my-workflow.yaml"

Validation passed. 4 steps, 0 approval gates, all variable references resolved.
```

This catches structural problems — missing fields, broken template variables, invalid step references — before they fail mid-execution.

## Key Features

### Sequential Steps with Context Flow

Each step's output becomes a variable that later steps can reference. This is how data moves through a recipe:

```yaml
name: "code-review"
version: "1.0.0"
description: "Multi-stage code review"
context:
  file_path: ""
steps:
  - id: "analyze"
    agent: "foundation:explorer"
    prompt: "Analyze the structure of {{file_path}}"
    output: "analysis"

  - id: "suggest"
    agent: "foundation:zen-architect"
    prompt: "Based on this analysis: {{analysis}}, suggest improvements"
    output: "suggestions"
```

### Agent, Bash, and Sub-Recipe Steps

Not every step needs an LLM. **Agent steps** (default) spawn a sub-agent with a prompt. **Bash steps** run shell commands directly — faster and deterministic. **Recipe steps** invoke another recipe as a sub-workflow:

```yaml
- id: "run-tests"
  type: "bash"
  command: "pytest --tb=short"
  output: "test_results"

- id: "security-audit"
  type: "recipe"
  recipe: "audits/security-scan.yaml"
  context:
    target: "{{file_path}}"
  output: "audit_results"
```

### Checkpointing and Resumability

Every completed step is checkpointed. If execution is interrupted, resume where you left off:

> Resume recipe session recipe_20260402_143022_a3f2

```
[Tool: recipes]
  operation: resume
  session_id: "recipe_20260402_143022_a3f2"

Resuming from step 3/5 [validate]...
```

### Approval Gates

Staged recipes pause at stage boundaries for human review — essential for deployments or data migrations:

```yaml
stages:
  - name: "planning"
    steps:
      - id: "generate-plan"
        agent: "foundation:zen-architect"
        prompt: "Plan the dependency upgrades"
        output: "upgrade_plan"

  - name: "execution"
    approval_required: true
    steps:
      - id: "apply-upgrades"
        agent: "foundation:modular-builder"
        prompt: "Apply these upgrades: {{upgrade_plan}}"
```

When execution reaches the approval gate, it pauses. You review the plan, then approve or deny:

> Approve the execution stage for the deploy session

```
[Tool: recipes]
  operation: approve
  session_id: "recipe_20260402_143022_a3f2"
  stage_name: "execution"
  message: "Plan looks good, proceed"
```

### Foreach Loops

Process collections by iterating over lists. Add `parallel: true` for concurrent execution:

```yaml
- id: "review-files"
  foreach: "{{files}}"
  as: "current_file"
  parallel: true
  collect: "all_reviews"
  agent: "foundation:zen-architect"
  prompt: "Review {{current_file}} for code quality"
```

### While Loops and Conditional Execution

For convergence workflows where iteration count isn't known upfront, use `while_condition`. For skipping steps based on runtime state, use `condition`:

```yaml
- id: "refine-draft"
  agent: "foundation:zen-architect"
  prompt: "Improve this draft: {{draft}}"
  output: "refined"
  while_condition: "{{quality_score}} < 8"
  max_while_iterations: 5

- id: "critical-fix"
  condition: "{{severity}} == 'critical'"
  agent: "foundation:modular-builder"
  prompt: "Apply critical fix to {{file_path}}"
```

## Example Recipes

The bundle includes working recipes across several domains:

| Recipe | What It Does |
|--------|-------------|
| **Code Review** | Analyze code structure, check security, generate improvement report |
| **Dependency Upgrade** | Audit dependencies, plan upgrades, validate, apply (with approval gates) |
| **Test Generation** | Analyze code paths, generate comprehensive tests, validate coverage |
| **Security Audit** | Multi-perspective security analysis from different threat angles |
| **Parallel Analysis** | Concurrent multi-file processing with collected results |

### The Authoring Lifecycle

Rather than writing YAML by hand, use the recipe-author agent to create recipes conversationally. Then use the result-validator to verify the output matches your intent.

> I need a recipe that reviews PRs by analyzing code changes, running security checks, validating test coverage, and requiring approval before posting comments

```
[Tool: task] Delegating to recipes:recipe-author
  └─ "Create a recipe for PR review with analysis, security, coverage, and approval gate"
```

The recipe-author asks clarifying questions — which agents to use, what context variables you need, whether steps should run in parallel — then generates valid YAML. Once you have a draft:
> Validate the recipe the author just created

```
[Tool: recipes]
  operation: validate
  recipe_path: "./recipes/pr-review.yaml"

Validation passed. 2 stages, 4 steps, 1 approval gate.
```

After execution, the result-validator checks whether the output actually accomplished your goal:
> Have the result-validator check whether the PR review recipe output matches my original requirements

```
[Tool: task] Delegating to recipes:result-validator
  └─ "Verify recipe output against original intent: PR review with analysis, security, coverage"
```

This create → validate → execute → verify cycle is the recommended way to develop recipes.

## Tips

**Start with flat recipes.** Use a simple `steps` list first. Add stages and approval gates only when your workflow genuinely needs human checkpoints.

**Validate before executing.** Always run `validate` on new or modified recipes. It catches problems that would otherwise fail mid-execution, wasting time and tokens.

**Use bash steps for deterministic work.** Running tests, installing dependencies, or fetching data with `curl` don't need an LLM. Bash steps are faster and more predictable.

**Name outputs descriptively.** Use `output: "security_findings"` rather than `output: "result"`. Later steps read more naturally: `{{security_findings}}` is self-documenting.

**Keep steps focused.** A step named "analyze-and-fix-and-report" is doing too much. Split it into three steps that pass context between them.

**Version your recipes.** Store recipe YAML in version control alongside your code. Bump the semver `version` field when you change the workflow.

**Test sub-recipes independently.** Sub-recipes receive only explicitly passed context, so each should work standalone. This makes debugging much faster.

## Next Steps

- **[Recipes Tool](../tools/recipes-tool.md)** — Full reference for all recipe operations
- **[Understanding Recipes](../concepts/recipes.md)** — Deeper dive into recipe concepts and patterns
- **[Foundation Bundle](./foundation.md)** — The agents commonly used in recipe steps
- **[Your First Bundle](../quickstart/first-bundle.md)** — How bundles work and how to install them
