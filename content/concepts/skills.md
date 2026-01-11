---
id: skills
type: concepts
title: "Understanding Skills"
---

# Understanding Skills

Skills are one of Amplifier's most powerful mechanisms for extending agent capabilities. They provide a way to package domain knowledge, best practices, and specialized workflows that agents can load on-demand.

## What is a Skill?

A skill is **loadable domain knowledge** - a markdown file containing instructions, patterns, examples, and references that an agent can incorporate into its context when needed.

Think of skills as "expertise modules" that agents can consult:

- **Domain expertise**: Coding standards, architectural patterns, security guidelines
- **Workflow guidance**: Step-by-step processes for complex tasks
- **Reference material**: API documentation, configuration schemas, examples
- **Best practices**: Industry standards, project conventions, quality guidelines

Skills differ from static documentation in a key way: they're designed to be **actively loaded into agent context** when relevant, rather than passively referenced.

### Skill vs. Context File

While both contain information, they serve different purposes:

| Aspect | Context Files | Skills |
|--------|---------------|--------|
| Loading | Always loaded at startup | Loaded on-demand |
| Scope | Session-wide context | Task-specific knowledge |
| Size | Should be concise | Can be comprehensive |
| Purpose | Shape agent behavior | Provide domain expertise |

## Discovering Skills

Before loading a skill, you need to know what's available. Amplifier provides discovery mechanisms through the `load_skill` tool.

### List All Available Skills

```python
load_skill(list=True)
```

This returns all skills discovered from configured directories, including:
- **Workspace skills**: `.amplifier/skills/` in your project
- **User skills**: `~/.amplifier/skills/` in your home directory
- **Collection skills**: Skills bundled with Amplifier collections

### Search for Skills

When you know roughly what you need but not the exact skill name:

```python
load_skill(search="python")
```

This filters skills by name or description matching your search term.

### Get Skill Metadata

Before loading a full skill (which consumes context), you can inspect its metadata:

```python
load_skill(info="python-standards")
```

This returns the skill's name, description, version, and path without loading the full content.

## Loading Skills

Once you've identified a relevant skill, load it to bring its knowledge into context.

### Basic Loading

```python
load_skill(skill_name="design-patterns")
```

This loads the complete skill content into the agent's context. The skill becomes part of the agent's working knowledge for the remainder of the task.

### What Happens When You Load a Skill

1. **Content injection**: The skill's markdown content is added to context
2. **Directory reference**: You receive a `skill_directory` path for accessing companion files
3. **Immediate availability**: The knowledge is immediately usable

### Accessing Companion Files

Many skills include companion files - examples, templates, or reference implementations:

```python
# Load the skill first
result = load_skill(skill_name="api-testing")

# Access companion files using the returned directory
read_file(result.skill_directory + "/examples/basic-test.py")
```

## Creating Skills

Skills are markdown files with a simple structure. Creating your own is straightforward.

### Skill File Structure

```markdown
---
name: my-custom-skill
description: Brief description of what this skill provides
version: 1.0.0
---

# Skill Title

## Overview
What this skill covers and when to use it.

## Guidelines
The actual domain knowledge, patterns, and practices.

## Examples
Concrete examples demonstrating the concepts.

## References
Links to additional resources or companion files.
```

### Where to Place Skills

Skills are discovered from these locations (in priority order):

1. **Workspace**: `.amplifier/skills/your-skill.md` - Project-specific skills
2. **User**: `~/.amplifier/skills/your-skill.md` - Personal skills across projects
3. **Collections**: Skills bundled with installed collections

First-match-wins: workspace skills override user skills with the same name.

### Skill Authoring Best Practices

**Be specific and actionable**
```markdown
# Good: Actionable guidance
When implementing retry logic:
1. Use exponential backoff starting at 100ms
2. Cap maximum retries at 5
3. Add jitter to prevent thundering herd

# Avoid: Vague advice
Retry logic should be implemented carefully.
```

**Include concrete examples**
```markdown
## Example: Rate Limiter Implementation

```python
class RateLimiter:
    def __init__(self, max_requests: int, window_seconds: int):
        self.max_requests = max_requests
        self.window = window_seconds
        self.requests = []
    
    def allow(self) -> bool:
        now = time.time()
        self.requests = [t for t in self.requests if now - t < self.window]
        if len(self.requests) < self.max_requests:
            self.requests.append(now)
            return True
        return False
```
```

**Reference companion files**
```markdown
## Templates

See the companion files for ready-to-use templates:
- `templates/api-client.py` - Base API client class
- `templates/retry-decorator.py` - Retry decorator implementation
```

### Skill Naming Conventions

- Use lowercase with hyphens: `python-standards`, `api-design`
- Be descriptive but concise: `react-testing` not `rt`
- Group related skills with prefixes: `aws-lambda`, `aws-s3`

## When to Use Skills

Skills shine in specific scenarios:

**Use skills when:**
- You need specialized domain knowledge for a task
- The knowledge is too large for always-on context files
- Multiple agents might need the same expertise
- You want to standardize approaches across a team

**Don't use skills when:**
- The information should always be available (use context files)
- You need real-time information (use web search)
- The task is simple and well-understood

## Skills vs. Other Knowledge Sources

| Source | Best For | Limitations |
|--------|----------|-------------|
| **Skills** | Domain expertise, patterns | Static, must be loaded |
| **Context files** | Always-needed guidance | Limited by context size |
| **Web search** | Current information | Variable quality |
| **Documentation** | Reference lookup | Not agent-optimized |

## Key Takeaways

1. **Skills are loadable expertise** - Domain knowledge packaged for agent consumption

2. **Discover before loading** - Use `list`, `search`, and `info` to find the right skill without wasting context

3. **Load on-demand** - Skills are task-specific; load them when needed, not preemptively

4. **Create your own** - Project or personal skills are simple markdown files in known locations

5. **Include companions** - Skills can reference example files, templates, and other resources

6. **Priority matters** - Workspace skills override user skills; first match wins

7. **Be specific** - Good skills provide actionable guidance and concrete examples, not vague advice

Skills transform agents from general-purpose assistants into domain experts. By packaging knowledge effectively, you enable agents to tackle specialized tasks with the same expertise a human specialist would bring.
