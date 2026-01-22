---
id: first-bundle
type: quickstart
title: "Your First Bundle"
---

# Your First Bundle

Bundles are the heart of the Amplifier ecosystem. They package skills, agents, recipes, and capabilities into reusable modules that extend your AI assistant's abilities. In this tutorial, you'll create a simple but practical bundle from scratch.

## What You'll Build

You'll create a **code-review** bundle that helps review Python code for common issues. This bundle will include:

- A custom agent that specializes in code review
- A skill document with Python best practices
- A recipe for automated code review workflow

By the end, you'll understand how to structure bundles, write agent instructions, and configure tools.

## Prerequisites

- Amplifier installed and configured
- Basic understanding of YAML format
- A text editor

## Step 1: Create Bundle Structure

Every bundle needs a specific directory structure. Let's create it:

```bash
mkdir -p ~/.amplifier/bundles/code-review
cd ~/.amplifier/bundles/code-review
```

Create the bundle manifest file `bundle.yaml`:

```yaml
name: code-review
version: 1.0.0
description: "Code review assistant for Python projects"
author: "Your Name"
license: MIT

# Define what this bundle provides
provides:
  agents:
    - code-reviewer
  skills:
    - python-best-practices
  recipes:
    - review-pr
```

This manifest tells Amplifier what your bundle provides and how to identify it.

### Create Directory Structure

Create the directories for your bundle components:

```bash
mkdir -p agents
mkdir -p skills
mkdir -p recipes
```

Your bundle structure should now look like:

```
~/.amplifier/bundles/code-review/
├── bundle.yaml
├── agents/
├── skills/
└── recipes/
```

## Step 2: Create Your Agent

Agents are specialized AI assistants with specific instructions. Create `agents/code-reviewer.yaml`:

```yaml
name: code-reviewer
description: "Reviews Python code for best practices and potential issues"

# Agent's system instructions
instructions: |
  You are an expert Python code reviewer. Your goal is to help developers
  write better, more maintainable code.

  When reviewing code:
  1. Check for PEP 8 style compliance
  2. Look for potential bugs or logic errors
  3. Identify security vulnerabilities
  4. Suggest performance improvements
  5. Recommend better naming or structure
  
  Always be constructive and educational in your feedback. Explain WHY
  something should change, not just WHAT to change.
  
  Format your reviews as:
  - **Issue**: Description of the problem
  - **Location**: File and line number
  - **Severity**: Critical / High / Medium / Low
  - **Recommendation**: Specific suggestion
  - **Explanation**: Why this matters

# Tools this agent can use
tools:
  - read_file
  - grep
  - glob
  - python_check
  - bash

# Optional: Tool-specific configuration
tool_config:
  python_check:
    checks: ["format", "lint", "types"]
```

### Understanding Agent Configuration

- **instructions**: The core prompt that defines the agent's behavior and expertise
- **tools**: Which capabilities the agent has access to
- **tool_config**: Optional settings for specific tools

## Step 3: Add Domain Knowledge with Skills

Skills provide domain-specific knowledge. Create `skills/python-best-practices.md`:

```markdown
---
name: python-best-practices
description: "Python coding standards and best practices"
version: 1.0.0
---

# Python Best Practices

## Code Style

Follow PEP 8 for consistent code style:
- Use 4 spaces for indentation (never tabs)
- Limit lines to 88 characters (Black formatter default)
- Use snake_case for functions and variables
- Use PascalCase for class names
- Use UPPER_CASE for constants

## Common Pitfalls

### Mutable Default Arguments

❌ **Bad:**
```python
def add_item(item, items=[]):
    items.append(item)
    return items
```

✅ **Good:**
```python
def add_item(item, items=None):
    if items is None:
        items = []
    items.append(item)
    return items
```

### Exception Handling

❌ **Bad:**
```python
try:
    risky_operation()
except:
    pass
```

✅ **Good:**
```python
try:
    risky_operation()
except ValueError as e:
    logger.error(f"Invalid value: {e}")
    raise
```

## Type Hints

Always use type hints for function signatures:

```python
def greet(name: str, age: int) -> str:
    return f"Hello {name}, you are {age} years old"
```

## Documentation

Use docstrings for all public modules, functions, classes, and methods:

```python
def calculate_total(items: list[float], tax_rate: float) -> float:
    """
    Calculate total price including tax.
    
    Args:
        items: List of item prices
        tax_rate: Tax rate as decimal (e.g., 0.08 for 8%)
    
    Returns:
        Total price including tax
        
    Raises:
        ValueError: If tax_rate is negative
    """
    if tax_rate < 0:
        raise ValueError("Tax rate cannot be negative")
    
    subtotal = sum(items)
    return subtotal * (1 + tax_rate)
```

## Testing

- Write tests for all public functions
- Aim for >80% code coverage
- Use pytest as the testing framework
- Keep tests isolated and independent

## Security

- Never hardcode secrets or credentials
- Validate all user input
- Use parameterized queries for SQL
- Keep dependencies updated
```

## Step 4: Create an Automated Recipe

Recipes define multi-step workflows. Create `recipes/review-pr.yaml`:

```yaml
name: review-pr
description: "Automated code review for pull requests"
version: 1.0.0

# Input context the recipe expects
context_schema:
  pr_number:
    type: integer
    description: "Pull request number to review"
    required: true

# Sequential workflow steps
steps:
  - name: fetch_changes
    agent: foundation:git-ops
    instruction: |
      Get the list of changed files in PR #{{pr_number}}.
      Return the file paths as a list.

  - name: review_code
    agent: code-review:code-reviewer
    instruction: |
      Review the following changed files: {{fetch_changes.result}}
      
      For each file:
      1. Read the file contents
      2. Check for style issues using python_check
      3. Review code quality and best practices
      4. Identify potential bugs
      
      Provide a comprehensive review report.

  - name: summarize
    agent: core:core-expert
    instruction: |
      Summarize the code review: {{review_code.result}}
      
      Create a concise summary suitable for posting as a PR comment,
      organized by severity level.
```

## Step 5: Test Your Bundle

First, verify your bundle is recognized:

```bash
cd ~/.amplifier/bundles/code-review
ls -la
```

You should see:
```
bundle.yaml
agents/code-reviewer.yaml
skills/python-best-practices.md
recipes/review-pr.yaml
```

### Test the Agent

Start Amplifier and try your agent:

```bash
amplifier
```

Then in the chat:

```
@code-reviewer Please review this Python function:

def calc(a, b):
    return a+b
```

Your agent should provide feedback about missing type hints, naming, and documentation.

### Test the Recipe

To test the recipe workflow:

```
Run recipe code-review:review-pr with pr_number=42
```

## Step 6: Refine and Iterate

Now that your bundle works, you can enhance it:

1. **Add more patterns** to the skill document
2. **Improve agent instructions** based on review quality
3. **Add tool configurations** for better defaults
4. **Create additional recipes** for different workflows

## Next Steps

Now that you've created your first bundle, explore:

- **[Bundle Architecture](../guides/bundle-architecture.md)** - Deep dive into bundle design
- **[Writing Effective Agents](../guides/writing-agents.md)** - Master agent instruction writing
- **[Recipe Development](../guides/recipes.md)** - Create complex automated workflows
- **[Skill Authoring](../guides/skills.md)** - Build comprehensive knowledge bases

### Share Your Bundle

Consider sharing your bundle with the community:

1. Publish to a Git repository
2. Add to the Amplifier bundle registry
3. Get feedback and contributions

## Troubleshooting

**Bundle not found**: Ensure `bundle.yaml` is in `~/.amplifier/bundles/code-review/`

**Agent not working**: Check that agent name in `bundle.yaml` matches the filename

**Recipe fails**: Verify all referenced agents exist and have required tools

## Summary

You've learned:

✅ How to structure a bundle with manifest, agents, skills, and recipes  
✅ How to write agent instructions and configure tools  
✅ How to create skill documents with domain knowledge  
✅ How to build automated workflows with recipes  
✅ How to test and refine your bundle  

Bundles are powerful because they're composable—your code-review bundle can be combined with other bundles to create even more sophisticated workflows.

Happy building! 🚀
