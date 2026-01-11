---
id: first-bundle
type: quickstart
title: "Your First Bundle"
---

# Your First Bundle

A bundle is a portable package of AI capabilities that you can share, compose, and reuse.
Think of it as a "plugin" that gives an AI assistant new skills, knowledge, and behaviors.

In this tutorial, you'll create a bundle that transforms any AI assistant into a
specialized code review helper.

## What You'll Build

By the end of this tutorial, you'll have:

- A complete bundle with custom instructions
- Configured tools for file operations
- Context files that guide behavior
- A working code reviewer you can use immediately

**Time required:** 10-15 minutes

**Prerequisites:**
- Amplifier installed and working
- Completed the "Hello World" quickstart
- A text editor

## Step 1: Create the Bundle Directory

Every bundle lives in its own directory with a specific structure. Let's create one:

```bash
# Create the bundle directory
mkdir -p ~/.amplifier/bundles/code-reviewer

# Navigate into it
cd ~/.amplifier/bundles/code-reviewer
```

The bundle directory name (`code-reviewer`) becomes the bundle's identifier.
You'll reference it as `code-reviewer` when composing with other bundles.

### Bundle Directory Structure

Here's what we'll create:

```
code-reviewer/
├── bundle.yaml          # Bundle manifest (required)
├── context/             # Context files loaded into AI
│   └── instructions.md  # Main behavior instructions
└── tools/               # Tool configurations (optional)
    └── tools.yaml       # Tool definitions
```

## Step 2: Create the Bundle Manifest

The `bundle.yaml` file defines your bundle's metadata and composition:

```bash
touch bundle.yaml
```

Add this content to `bundle.yaml`:

```yaml
# Bundle manifest for code-reviewer
name: code-reviewer
version: 1.0.0
description: "A focused code review assistant that provides actionable feedback"

# What this bundle provides
provides:
  - code-review
  - best-practices

# Context files to load (order matters)
context:
  - context/instructions.md

# Tools this bundle needs
tools:
  - read_file
  - grep
  - glob

# Optional: bundles this one extends
extends: []
```

### Understanding the Manifest

| Field | Purpose |
|-------|---------|
| `name` | Unique identifier for your bundle |
| `version` | Semantic version for tracking changes |
| `description` | Human-readable explanation |
| `provides` | Capabilities this bundle offers |
| `context` | Files loaded into the AI's context |
| `tools` | Tools the AI can use |
| `extends` | Other bundles to inherit from |

## Step 3: Add Instructions

The instructions file is where you define your bundle's personality and behavior.
This is the most important file—it shapes how the AI acts.

Create the context directory and instructions file:

```bash
mkdir -p context
touch context/instructions.md
```

Add this content to `context/instructions.md`:

```markdown
# Code Review Assistant

You are a focused code review assistant. Your job is to provide clear,
actionable feedback that helps developers write better code.

## Review Philosophy

- **Be specific**: Point to exact lines and explain why something matters
- **Be constructive**: Suggest improvements, don't just criticize
- **Be prioritized**: Distinguish critical issues from nice-to-haves
- **Be educational**: Explain the "why" behind recommendations

## Review Process

When asked to review code:

1. **Understand context**: What does this code do? What problem does it solve?
2. **Check correctness**: Does it work? Are there bugs or edge cases?
3. **Evaluate clarity**: Is it readable? Would a new team member understand it?
4. **Assess maintainability**: Is it easy to change? Are there hidden dependencies?
5. **Consider performance**: Are there obvious inefficiencies?

## Response Format

Structure your reviews like this:

### Summary
One paragraph overview of the code and your overall assessment.

### Critical Issues
Problems that must be fixed (bugs, security issues, data loss risks).

### Improvements
Suggestions that would meaningfully improve the code.

### Minor Notes
Style suggestions, nitpicks, or optional enhancements.

## What NOT to Do

- Don't rewrite entire files unprompted
- Don't focus on style over substance
- Don't be vague ("this could be better")
- Don't overwhelm with too many comments

## Example Interaction

User: "Review this function for me"

Good response:
- Start with what the function does
- Identify the most important issue first
- Provide a specific fix with explanation
- Mention 1-2 secondary concerns
- End with what's done well

Bad response:
- List 20 minor style issues
- Rewrite the entire function
- Give generic advice without specifics
```

## Step 4: Configure Tools

Your bundle can specify which tools it needs. Create a tools configuration:

```bash
mkdir -p tools
touch tools/tools.yaml
```

Add this content to `tools/tools.yaml`:

```yaml
# Tools configuration for code-reviewer bundle

# File reading for examining code
read_file:
  enabled: true
  description: "Read source files to review"

# Pattern searching for finding issues
grep:
  enabled: true
  description: "Search for patterns in code"

# File discovery
glob:
  enabled: true
  description: "Find files to review"

# Tools we explicitly don't need
write_file:
  enabled: false
  reason: "Review only - no modifications"

bash:
  enabled: false
  reason: "Review only - no execution"
```

### Why Limit Tools?

Restricting tools creates a safer, more focused assistant:

- **Principle of least privilege**: Only enable what's needed
- **Clear boundaries**: Users know what the assistant can/can't do
- **Reduced risk**: No accidental file modifications or command execution

## Step 5: Test Your Bundle

Now let's verify everything works. First, validate the bundle structure:

```bash
# Check that all files exist
ls -la ~/.amplifier/bundles/code-reviewer/
```

You should see:

```
bundle.yaml
context/
  instructions.md
tools/
  tools.yaml
```

### Run with Your Bundle

Start Amplifier with your new bundle:

```bash
amp --bundle code-reviewer
```

Or add it to an existing configuration:

```bash
amp --bundle foundation --bundle code-reviewer
```

### Test the Behavior

Try these prompts to verify it's working:

```
Review this Python function:

def get_user(id):
    users = load_all_users()
    for u in users:
        if u.id == id:
            return u
    return None
```

Your code reviewer should:
1. Identify the inefficiency (loading all users)
2. Suggest using a dictionary lookup or database query
3. Note the missing error handling
4. Format the response according to your instructions

## Step 6: Iterate and Improve

Your first bundle is working! Now refine it based on usage:

### Add More Context

Create additional context files for specific scenarios:

```bash
# Security-focused review guidelines
touch context/security-review.md

# Performance review checklist
touch context/performance-review.md
```

Update `bundle.yaml` to include them:

```yaml
context:
  - context/instructions.md
  - context/security-review.md
  - context/performance-review.md
```

### Create Specialized Variants

Make language-specific versions:

```bash
mkdir -p ~/.amplifier/bundles/python-reviewer
# Copy and customize for Python-specific guidance
```

### Compose with Other Bundles

Extend existing bundles to add capabilities:

```yaml
# In a new bundle's bundle.yaml
extends:
  - code-reviewer
  - python-standards
```

## Troubleshooting

### Bundle Not Loading

Check these common issues:

1. **File location**: Bundle must be in `~/.amplifier/bundles/` or a configured path
2. **YAML syntax**: Validate with `yamllint bundle.yaml`
3. **File permissions**: Ensure files are readable

### Instructions Not Taking Effect

- Context files must be listed in `bundle.yaml`
- Check file paths are relative to bundle root
- Verify no syntax errors in markdown

### Tools Not Available

- Tools must be enabled in the base configuration
- Bundle can only restrict, not add new tools
- Check tool names match exactly

## Next Steps

You've created your first bundle. Here's where to go next:

1. **[Bundle Composition](./bundle-composition.md)**: Learn to combine bundles effectively
2. **[Advanced Context](./advanced-context.md)**: Dynamic context and conditional loading
3. **[Publishing Bundles](./publishing-bundles.md)**: Share your bundles with others
4. **[Bundle Patterns](../patterns/bundle-patterns.md)**: Common patterns and best practices

## Quick Reference

### Bundle Checklist

- [ ] `bundle.yaml` with name, version, description
- [ ] At least one context file with instructions
- [ ] Tools configured appropriately
- [ ] Tested with real prompts
- [ ] Documentation for users

### Key Commands

```bash
# List available bundles
amp bundles list

# Validate bundle structure
amp bundles validate code-reviewer

# Run with bundle
amp --bundle code-reviewer

# Combine bundles
amp --bundle foundation --bundle code-reviewer
```

---

Congratulations! You've built a reusable, composable AI capability.
The bundle pattern lets you capture expertise once and apply it everywhere.
