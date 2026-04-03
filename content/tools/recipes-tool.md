---
id: tool-recipes
type: tool
title: "Recipes Tool"
---

# Recipes Tool

The delegate tool lets you hand off a single task to a sub-agent. But what about multi-step workflows that need to happen in a specific order, with checkpoints, approvals, and the ability to resume if something goes wrong? That's what the recipes tool does — it executes declarative YAML workflows that define entire pipelines of agent work.

Think of recipes as saved procedures. Instead of manually prompting "first analyze, then review, then fix, then test" every time, you encode that sequence once in YAML and run it whenever you need it. Amplifier handles the execution order, state persistence, and error recovery.

## Core Capabilities

The recipes tool supports eight operations:

- **execute** — run a recipe from a YAML file, with optional context variables
- **resume** — continue an interrupted session by its session ID
- **validate** — check a recipe's YAML structure before running it
- **list** — show all active recipe sessions
- **approvals** — list pending approval gates across all sessions
- **approve** — approve a stage to let execution continue (with an optional message)
- **deny** — deny a stage to stop execution (with an optional reason)
- **cancel** — cancel a running session, either gracefully or immediately

The first three — execute, validate, resume — cover the basic lifecycle. The approval operations add human-in-the-loop control. And cancel is your emergency brake.

## Executing a Recipe

The most common operation. Point the tool at a YAML file and optionally pass context variables:

> Run the code review recipe on the auth module

```
[Tool: recipes]
  operation: execute
  recipe_path: "@recipes:code-review.yaml"
  context: { "file_path": "src/auth.py" }

Session started: recipe_20260402_091544_b7e3
Step 1/2 [analyze]: Analyzing src/auth.py for code quality...
Step 2/2 [suggest]: Generating improvement suggestions...
Recipe completed successfully.
```

The `@bundle:path` format references recipes from installed bundles — you don't need to know the absolute path. Context variables become available inside the recipe as `{{ file_path }}`, `{{ environment }}`, or whatever keys you pass.

> Validate the deploy recipe before I run it

```
[Tool: recipes]
  operation: validate
  recipe_path: "./recipes/deploy.yaml"

Validation passed. 2 stages, 4 steps, 1 approval gate.
```

Always validate new or modified recipes before executing. It catches structural problems — missing required fields, invalid step references, broken template variables — before they fail mid-execution.

## Managing Sessions

Every recipe execution creates a session. Sessions track progress, store intermediate results, and enable resumption:

> Show my active recipe sessions

```
[Tool: recipes]
  operation: list

Active sessions:
  recipe_20260402_091544_b7e3  code-review      completed   2m ago
  recipe_20260402_084312_c1a9  deploy-staging   paused      18m ago (awaiting approval)
```

If a session gets interrupted — network issue, timeout, or you just close your laptop — you can pick up where it left off:

> Resume the deploy session

```
[Tool: recipes]
  operation: resume
  session_id: "recipe_20260402_084312_c1a9"

Resuming session recipe_20260402_084312_c1a9...
Restored checkpoint at stage 'execution', step 2/3.
Continuing from step [run-migrations]...
```

The recipe resumes from the last successful checkpoint, not from the beginning. Steps that already completed don't re-run.

And if something goes wrong and you need to stop a recipe entirely:

> Cancel the deploy session

```
[Tool: recipes]
  operation: cancel
  session_id: "recipe_20260402_084312_c1a9"
  immediate: false

Session recipe_20260402_084312_c1a9 cancelled gracefully.
Current step allowed to finish before stopping.
```

Setting `immediate: true` kills the current step mid-execution. Graceful cancellation (the default) lets the running step finish, then stops.

## Approval Gates

This is where recipes go beyond simple automation. Staged recipes pause at designated points and wait for a human to review and approve before continuing. Any step that modifies production systems, sends notifications, or makes irreversible changes should sit behind an approval gate.

> Check if any recipes are waiting for my approval

```
[Tool: recipes]
  operation: approvals

Pending approvals:
  recipe_20260402_084312_c1a9  deploy-staging
    Stage: production-deploy
    Waiting since: 18 minutes ago
    Planning output available for review.
```

> Approve the production deploy — go ahead and merge

```
[Tool: recipes]
  operation: approve
  session_id: "recipe_20260402_084312_c1a9"
  stage_name: "production-deploy"
  message: "merge"

Stage 'production-deploy' approved. Resuming execution...
Step [deploy]: Deploying to production...
```

The optional `message` parameter passes information to subsequent steps — the recipe can read it as `{{ _approval_message }}`. In this case, the deploy step knows you said "merge" and can act accordingly.

If the plan doesn't look right, deny it:

> Deny the production deploy — the migration plan needs revision

```
[Tool: recipes]
  operation: deny
  session_id: "recipe_20260402_084312_c1a9"
  stage_name: "production-deploy"
  reason: "Migration plan needs revision — missing rollback step"

Stage 'production-deploy' denied. Execution stopped.
Reason recorded: Migration plan needs revision — missing rollback step
```

Denial stops the recipe. You can fix the issue and re-execute from scratch, or modify the recipe and try again.

## Recipes vs. Direct Delegation

The delegate tool and the recipes tool overlap in purpose — both orchestrate agent work. Here's when to reach for each:

| Situation | Use | Why |
|-----------|-----|-----|
| One-off investigation | **Delegate** | No need for YAML overhead |
| Repeatable workflow you run weekly | **Recipes** | Define once, run many times |
| Single agent, single task | **Delegate** | Simpler, faster |
| Multi-step pipeline with dependencies | **Recipes** | Steps chain results automatically |
| Needs human approval mid-process | **Recipes** | Approval gates are built in |
| Parallel independent analyses | **Delegate** | Launch multiple agents simultaneously |
| Must survive interruptions | **Recipes** | Checkpointing and resume built in |
| Quick prototype of a workflow | **Delegate** | Iterate in conversation first, formalize as recipe later |

The natural progression: start by doing things manually with delegate, notice you're repeating the same sequence, then encode it as a recipe. Recipes are for workflows that have graduated from "let me try this" to "we do this regularly."

## Tips

- **Validate before executing.** New recipes frequently have template variable typos or missing step IDs. A quick validate catches these instantly.
- **Pass context, don't hardcode.** Use `{{ variable }}` templates instead of embedding file paths or environment names directly in the YAML. The same recipe should work across projects.
- **Use approval gates for anything irreversible.** Deploys, data migrations, external API calls — if you can't undo it, gate it.
- **Keep steps focused.** Each step should do one thing well. "Analyze code quality" is a good step. "Analyze, refactor, test, and deploy" is four steps crammed into one.
- **Start simple.** A recipe with two steps and no approval gates is a perfectly valid recipe. Add complexity only when you need it.
- **Use the approval message.** When approving a stage, pass a message like `"merge"` or `"pr"` to influence how subsequent steps behave — recipes can branch on `{{ _approval_message }}`.
- **Cancel gracefully by default.** Immediate cancellation can leave state in a messy condition. Use `immediate: true` only when you need to stop *right now*.

## Next Steps

- See the [Delegate Tool](./task.md) for one-off agent delegation without YAML overhead
- Learn about [Recipes as a concept](../concepts/recipes.md) for how recipe YAML is structured in detail
- Explore [Bash Tool](./bash.md) for the shell commands that recipe steps often invoke internally
