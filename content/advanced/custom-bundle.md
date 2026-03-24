---
id: custom-bundle
type: advanced
title: "Creating Custom Bundles"
---

# Creating Custom Bundles

Build your own bundle to customize Amplifier for your workflow.

## Overview

A custom bundle lets you:

- **Combine existing bundles** - Mix and match capabilities
- **Add custom agents** - Specialists for your domain
- **Include context** - Project-specific knowledge
- **Set defaults** - Provider, model, instructions

## Bundle Structure

```
my-bundle/
├── bundle.yaml          # Main configuration
├── agents/              # Custom agents
│   └── my-agent.yaml
├── behaviors/           # Reusable capabilities
│   └── my-behavior.yaml
├── context/             # Knowledge files
│   └── project-info.md
└── modules/             # Custom tools (optional)
    └── my-tool/
```

## Basic Bundle

### Minimal Example

```yaml
# my-bundle/bundle.yaml
bundle:
  name: my-custom
  version: 1.0.0
  description: My custom Amplifier configuration

includes:
  - bundle: foundation

instructions: |
  You are a helpful assistant for my project.
  Always be concise and direct.
```

### Use It

```bash
amp --bundle ./my-bundle
```

## Including Other Bundles

### Stack Multiple Bundles

```yaml
includes:
  - bundle: foundation       # Core tools and agents
  - bundle: recipes          # Workflow orchestration
  - bundle: lsp-python       # Python code intelligence
```

### From GitHub

```yaml
includes:
  - bundle: git+https://github.com/microsoft/amplifier-bundle-recipes@main
  - bundle: git+https://github.com/robotdad/amplifier-bundle-lsp-typescript@main
```

### From Local Path

```yaml
includes:
  - bundle: ../shared-bundle
  - bundle: ~/my-bundles/common
```

## Custom Agents

### Define an Agent

```yaml
# agents/api-expert.yaml
meta:
  name: api-expert
  description: "REST API design and review specialist"

# Which tools this agent can use
tools:
  - read_file
  - grep
  - web_search

# Agent's specialized knowledge
instructions: |
  You are a REST API expert.
  
  When reviewing APIs:
  - Check resource naming conventions
  - Verify HTTP method usage
  - Ensure consistent error responses
  - Look for proper versioning
  
  Reference OpenAPI spec and REST best practices.

# Optional: load additional context
context:
  include:
    - ../context/api-guidelines.md
```

### Register in Bundle

```yaml
# bundle.yaml
agents:
  - path: ./agents/api-expert.yaml
  - path: ./agents/db-expert.yaml
```

### Use Your Agent

```
> Use api-expert to review our user endpoints
```

## Custom Context

### Add Knowledge Files

```yaml
# bundle.yaml
context:
  include:
    - ./context/project-overview.md
    - ./context/coding-standards.md
    - ./context/architecture.md
```

### Context File Example

```markdown
# context/project-overview.md

## Project: Acme API

This is a REST API for the Acme product catalog.

### Key Components

- **Products Service** - CRUD for products
- **Orders Service** - Order processing
- **Auth Service** - JWT authentication

### Coding Standards

- Python 3.12+
- Type hints required
- Docstrings in Google style
- Tests required for new code
```

## Custom Instructions

### Bundle-Level Instructions

```yaml
# bundle.yaml
instructions: |
  You are an assistant for the Acme project.
  
  Key guidelines:
  - Follow our coding standards in context/coding-standards.md
  - Always suggest type hints
  - Recommend tests for new code
  - Use our error handling patterns
```

### Layer with Includes

Your instructions add to included bundles:

```
Foundation instructions (base)
     ↓
Your bundle instructions (added)
     ↓
Final combined instructions
```

## Behaviors

Behaviors are reusable agent + context combinations:

### Define a Behavior

```yaml
# behaviors/code-review.yaml
meta:
  name: code-review
  description: "Thorough code review process"

# What this behavior adds
context:
  include:
    - ./review-checklist.md

# Specialized agent config
agent:
  instructions: |
    When reviewing code:
    1. Check for bugs and logic errors
    2. Review naming and readability
    3. Assess test coverage
    4. Look for security issues
```

### Use in Bundle

```yaml
behaviors:
  - path: ./behaviors/code-review.yaml
  - path: ./behaviors/security-audit.yaml
```

## Provider Configuration

### Set Default Provider

```yaml
# bundle.yaml
providers:
  - module: provider-anthropic
    config:
      model: claude-sonnet-4-20250514
      max_tokens: 8192
```

### Multiple Providers

```yaml
providers:
  - module: provider-anthropic
    config:
      model: claude-sonnet-4-20250514
    default: true  # Use this by default
      
  - module: provider-ollama
    config:
      model: llama3.1
```

## Tools Configuration

### Add Tools

```yaml
tools:
  - module: tool-filesystem
  - module: tool-bash
    config:
      allowed_commands: ["ls", "cat", "grep", "git"]
```

### Custom Tool Module

```yaml
tools:
  - module: jira-tool
    source: ./modules/jira-tool
    config:
      api_url: https://acme.atlassian.net
```

## Complete Example

```yaml
# bundle.yaml
bundle:
  name: acme-dev
  version: 1.0.0
  description: Acme project development bundle

# Build on existing bundles
includes:
  - bundle: foundation
  - bundle: recipes
  - bundle: lsp-python

# Default provider
providers:
  - module: provider-anthropic
    config:
      model: claude-sonnet-4-20250514

# Custom agents
agents:
  - path: ./agents/api-expert.yaml
  - path: ./agents/db-expert.yaml

# Behaviors
behaviors:
  - path: ./behaviors/code-review.yaml

# Project knowledge
context:
  include:
    - ./context/project-overview.md
    - ./context/coding-standards.md

# Custom instructions
instructions: |
  You are an Acme project assistant.
  
  Guidelines:
  - Follow our coding standards
  - Always suggest type hints
  - Write tests for new code
  - Use project patterns from context
```

## Try It Yourself

### Exercise 1: Create Minimal Bundle

```bash
mkdir my-bundle
cat > my-bundle/bundle.yaml << 'EOF'
bundle:
  name: my-test
  version: 1.0.0

includes:
  - bundle: foundation

instructions: |
  Be concise. Use bullet points.
EOF

amp --bundle ./my-bundle
```

### Exercise 2: Add an Agent

Create `my-bundle/agents/reviewer.yaml`:
```yaml
meta:
  name: reviewer
  description: "Code reviewer"

instructions: |
  Review code for bugs and improvements.
  Be specific and actionable.
```

Update bundle.yaml:
```yaml
agents:
  - path: ./agents/reviewer.yaml
```

Test:
```
> Use reviewer to check src/main.py
```

### Exercise 3: Add Context

Create `my-bundle/context/standards.md` with your coding standards.

Update bundle.yaml:
```yaml
context:
  include:
    - ./context/standards.md
```

## Sharing Bundles

### Via Git

```bash
# Push to GitHub
cd my-bundle
git init
git add .
git commit -m "Initial bundle"
gh repo create my-bundle --public
git push -u origin main
```

Others can use:
```bash
amp bundle add git+https://github.com/yourusername/my-bundle@main
```

### Via Zip

```bash
zip -r my-bundle.zip my-bundle/
# Share the zip file
```
