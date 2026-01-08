---
id: first-bundle
type: quickstart
title: "Your First Bundle"
---

# Your First Bundle

Bundles extend Amplifier with new capabilities. Let's add the **recipes** bundle to unlock multi-step workflows.

## What is a Bundle?

A bundle is a package that configures Amplifier with:

- **Tools** - New capabilities (e.g., LSP for code navigation)
- **Agents** - Specialized AI configurations (e.g., security reviewer)
- **Behaviors** - Reusable patterns and context
- **Instructions** - System prompts that guide the AI

Think of bundles like browser extensions - they add features without changing the core.

## Installing the Recipes Bundle

The recipes bundle adds the ability to run multi-step, declarative workflows.

```bash
# Add the bundle
amplifier bundle add git+https://github.com/microsoft/amplifier-bundle-recipes@main

# Activate it
amplifier bundle use recipes

# Verify it's loaded
amplifier run "/tools" | grep recipe
```

You should see the `recipes` tool is now available.

## What Recipes Can Do

Recipes are YAML files that define multi-step workflows:

```yaml
# example-recipe.yaml
name: code-review
description: Review code for quality and security

steps:
  - id: analyze
    agent: foundation:zen-architect
    instruction: "Analyze {{file}} for design issues"
    
  - id: security
    agent: foundation:security-guardian  
    instruction: "Check {{file}} for security vulnerabilities"
    
  - id: summarize
    instruction: |
      Create a review summary from:
      - Design analysis: {{analyze.result}}
      - Security findings: {{security.result}}
```

Key features:
- **Multiple agents** - Each step can use a different specialist
- **Context flow** - Results from earlier steps available to later ones
- **Variables** - `{{file}}` is passed in when you run the recipe
- **Resumable** - If interrupted, pick up where you left off

## Running Your First Recipe

The recipes bundle comes with example recipes. Let's run one:

```bash
# See available example recipes
ls ~/.amplifier/bundles/recipes/examples/

# Run a code review recipe
amplifier recipes execute recipes/code-review.yaml \
  --context '{"file_path": "src/main.py"}'
```

Or explore recipes interactively:

```bash
amplifier

# In the session:
amplifier> What recipes are available?
amplifier> Run the code-review recipe on src/auth.py
```

## Creating a Simple Recipe

Create your own recipe:

```bash
# Create a recipes directory in your project
mkdir -p .amplifier/recipes
```

```yaml
# .amplifier/recipes/explain-and-test.yaml
name: explain-and-test
description: Explain code and generate tests

context:
  file: ""  # Required: path to file

steps:
  - id: explain
    instruction: |
      Explain what {{file}} does in plain language.
      Focus on the main purpose and key functions.

  - id: generate-tests
    agent: foundation:modular-builder
    instruction: |
      Based on this explanation:
      {{explain.result}}
      
      Generate comprehensive tests for {{file}}.
      Include edge cases and error conditions.
      
  - id: save-tests
    instruction: |
      Save the tests to tests/test_{{file | basename}}.
      
      Tests to save:
      {{generate-tests.result}}
```

Run it:

```bash
amplifier recipes execute .amplifier/recipes/explain-and-test.yaml \
  --context '{"file": "src/utils.py"}'
```

## Recipes with Approval Gates

Add human checkpoints for critical operations:

```yaml
# deploy-recipe.yaml
name: safe-deploy

steps:
  - id: build
    instruction: "Build the project and run tests"
    
  - id: review
    instruction: "Summarize changes since last deploy"
    
  - id: deploy
    requires_approval: true  # Pauses here for human approval
    instruction: "Deploy to production"
```

When you run this, it will pause at the deploy step:

```
Step 'deploy' requires approval.
To approve: amplifier recipes approve [session-id] deploy
To deny: amplifier recipes deny [session-id] deploy
```

## Other Popular Bundles

| Bundle | What it adds |
|--------|-------------|
| `recipes` | Multi-step workflows |
| `lsp-python` | Python code intelligence (go-to-definition, find references) |
| `lsp-typescript` | TypeScript/JS code intelligence |
| `design-intelligence` | 7 specialized design agents |

Install any bundle:

```bash
# Python code intelligence
amplifier bundle add git+https://github.com/microsoft/amplifier-bundle-lsp-python@main
amplifier bundle use lsp-python

# Now you have semantic code navigation
amplifier run "Find all usages of the authenticate() function"
```

## Try It Yourself

1. **Install recipes**: Follow the steps above to add the recipes bundle
2. **Run an example**: Try the code-review recipe on one of your files
3. **Create your own**: Write a simple recipe for a task you do often

## What's Next?

You've completed the Quick Start! You can now:

- **[Core Concepts](../concepts/index.md)** - Understand bundles, modules, and agents in depth
- **[Tools Reference](../tools/index.md)** - Explore all available tools
- **[Developer Setup](../dev-setup/index.md)** - Professional development workflows

---

## Quick Start Summary

You've learned:

1. **What Amplifier is** - A modular AI agent framework
2. **How it's different** - Multi-provider, extensible, transparent
3. **How to install** - UV + one command
4. **Basic usage** - Interactive and single-shot modes
5. **Key commands** - CLI and in-session commands
6. **Bundles** - Extending capabilities with recipes

You're ready to explore deeper. Welcome to Amplifier!
