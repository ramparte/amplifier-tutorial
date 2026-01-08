---
id: advanced-overview
type: advanced
title: "Advanced Topics"
---

# Advanced Topics

Extend and customize Amplifier for your specific needs.

## Overview

| Topic | What You'll Learn |
|-------|-------------------|
| [Custom Bundles](custom-bundle.md) | Create your own configuration packages |
| [Custom Tools](custom-tool.md) | Build new capabilities |
| [Custom Recipes](custom-recipe.md) | Design multi-step workflows |
| [MCP Integration](mcp-integration.md) | Connect to MCP servers |

## Customization Levels

From simple to complex:

```
1. Instructions only    → Add instructions to existing bundle
2. Custom bundle        → Combine bundles, add agents/context
3. Custom tools         → New capabilities via Python
4. Custom recipes       → Multi-step workflow orchestration
5. MCP integration      → Connect to external systems
```

## Quick Decision Guide

| I Want To... | Use |
|--------------|-----|
| Change AI behavior | [Custom Bundle](custom-bundle.md) → instructions |
| Add project knowledge | [Custom Bundle](custom-bundle.md) → context |
| Create specialists | [Custom Bundle](custom-bundle.md) → agents |
| Add new capabilities | [Custom Tool](custom-tool.md) |
| Automate multi-step work | [Custom Recipe](custom-recipe.md) |
| Connect to databases/APIs | [MCP Integration](mcp-integration.md) |

## Common Patterns

### Pattern 1: Team Configuration

Create a bundle for your team:

```yaml
# team-bundle/bundle.yaml
bundle:
  name: acme-team
  
includes:
  - bundle: foundation
  - bundle: recipes
  
agents:
  - path: ./agents/code-reviewer.yaml
  
context:
  include:
    - ./context/coding-standards.md
    - ./context/architecture.md
```

### Pattern 2: Project Workflows

Create recipes for project processes:

```yaml
# recipes/deploy.yaml
name: deploy
steps:
  - id: test
    instruction: "Run tests"
  - id: approve
    requires_approval: true
  - id: deploy
    instruction: "Deploy to staging"
```

### Pattern 3: Integration Extension

Add tools for your systems:

```python
# modules/jira_tool/tool.py
class JiraTool(Tool):
    name = "jira"
    description = "Manage Jira tickets"
    
    async def execute(self, input):
        # Your Jira integration
        pass
```

### Pattern 4: External Data

Connect via MCP:

```yaml
mcp:
  servers:
    - name: internal-api
      command: python internal_api_server.py
```

## Starting Points

### Minimal Custom Bundle

```bash
mkdir my-bundle
cat > my-bundle/bundle.yaml << 'EOF'
bundle:
  name: my-config
  version: 1.0.0
includes:
  - bundle: foundation
instructions: |
  My custom instructions here.
EOF

amp --bundle ./my-bundle
```

### Minimal Custom Tool

```python
# tool.py
from amplifier_core import Tool

class MyTool(Tool):
    name = "my-tool"
    description = "Does something useful"
    input_schema = {"type": "object", "properties": {}}
    
    async def execute(self, input):
        return "Result"
```

### Minimal Recipe

```yaml
name: my-recipe
steps:
  - id: step-one
    instruction: "Do the thing"
```

## Best Practices

### Start Simple

1. First, try adjusting instructions
2. Then, add context files
3. Then, create agents
4. Finally, build tools/recipes

### Version Control

```bash
# Keep your customizations in git
cd my-bundle
git init
git add .
git commit -m "Initial bundle"
```

### Documentation

Document your customizations:

```
my-bundle/
├── bundle.yaml
├── README.md          # What this bundle does
├── agents/
│   └── README.md      # What each agent does
└── recipes/
    └── README.md      # How to use recipes
```

### Testing

Test changes incrementally:

```bash
# Test bundle loads
amp --bundle ./my-bundle
> What tools do you have?

# Test specific agent
> Use [your-agent] to do [task]

# Test recipe
amp recipes validate my-recipe.yaml
amp recipes execute my-recipe.yaml --context '{}'
```

## Need Help?

- **Recipe creation** → Ask `recipe-author` agent
- **Architecture questions** → Ask `zen-architect` agent
- **Debug issues** → Check [Debugging guide](../dev-setup/debugging.md)
