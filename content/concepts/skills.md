---
id: skills
type: concepts
title: "Understanding Skills"
---

# Understanding Skills

Skills are one of Amplifier's most powerful features for extending agent capabilities. They provide a way to inject domain-specific knowledge, best practices, and workflows into your agents without modifying core system code.

## What is a Skill?

A skill is **loadable domain knowledge** packaged as markdown files that agents can access on demand. Think of skills as specialized instruction manuals that agents can consult when they need expertise in a particular area.

### Key Characteristics

- **On-demand loading**: Skills are loaded only when needed, keeping agent context lean
- **Domain-specific**: Each skill focuses on a particular area of expertise
- **Declarative**: Skills describe *what* to do, not *how* to implement it
- **Composable**: Multiple skills can be loaded together for complex tasks
- **Versionable**: Skills can be versioned and updated independently

### Anatomy of a Skill

A skill typically consists of:

```
my-skill/
├── skill.md          # Main skill content (required)
├── examples/         # Optional companion files
│   ├── basic.py
│   └── advanced.py
└── templates/        # Optional templates
    └── config.yaml
```

The `skill.md` file contains:

1. **Frontmatter**: Metadata about the skill (name, version, description)
2. **Context**: Background knowledge the agent needs
3. **Guidelines**: Best practices and decision frameworks
4. **Workflows**: Step-by-step procedures for common tasks
5. **Examples**: Concrete illustrations of concepts

## Discovering Skills

Before loading a skill, you need to know what's available. Amplifier provides several ways to discover skills.

### Listing All Skills

Use the `load_skill` tool with the `list` parameter:

```python
load_skill(list=True)
```

This returns all available skills with their names and descriptions.

### Searching for Skills

If you're looking for skills in a specific domain:

```python
load_skill(search="python")
```

This filters skills by name or description matching your search term.

### Getting Skill Metadata

Before loading a full skill (which consumes context), you can inspect its metadata:

```python
load_skill(info="design-patterns")
```

This returns the skill's name, description, version, and path without loading the full content.

### Skill Discovery Locations

Skills are discovered from multiple directories in priority order:

1. **Workspace skills** (`.amplifier/skills/`) - Project-specific skills
2. **User skills** (`~/.amplifier/skills/`) - Personal skills across projects
3. **Collection skills** - Skills bundled with Amplifier collections

First-match-wins: if the same skill exists in multiple locations, the first one found is used.

## Loading Skills

When you need domain expertise, load the relevant skill into context.

### Basic Loading

```python
load_skill(skill_name="python-standards")
```

This loads the full skill content and returns:
- The skill content itself
- The `skill_directory` path for accessing companion files

### Accessing Companion Files

Many skills include additional resources like examples, templates, or reference files. Use the returned `skill_directory` to access them:

```python
# Load the skill
result = load_skill(skill_name="api-design")
skill_dir = result["skill_directory"]

# Read a companion file
read_file(f"{skill_dir}/examples/rest-api.py")
```

### When to Load Skills

Load skills when you need:

- **Domain expertise**: Working in an unfamiliar area
- **Best practices**: Ensuring code follows established patterns
- **Consistency**: Applying the same standards across a project
- **Complex workflows**: Following multi-step procedures correctly

### Context Management

Skills consume context tokens when loaded. Best practices:

- Load skills on-demand, not preemptively
- Use `info` to check skill size before loading
- Load only the skills you actually need
- Consider skill size when working with limited context

## Creating Skills

You can create custom skills for your projects or personal use.

### Skill Structure

Create a directory with at least a `skill.md` file:

```markdown
---
name: my-custom-skill
description: Brief description of what this skill provides
version: 1.0.0
---

# My Custom Skill

## Overview
Explain the domain this skill covers.

## Guidelines
List best practices and decision frameworks.

## Workflows
Document step-by-step procedures.

## Examples
Provide concrete illustrations.
```

### Where to Place Skills

Choose based on scope:

| Location | Scope | Use Case |
|----------|-------|----------|
| `.amplifier/skills/` | Project | Team-specific patterns |
| `~/.amplifier/skills/` | User | Personal workflows |
| Collection | Distribution | Shared with community |

### Writing Effective Skills

**Be specific**: Focus on one domain or problem area.

```markdown
# Good: Focused skill
---
name: react-testing
description: Testing patterns for React components
---

# Bad: Too broad
---
name: frontend
description: Everything about frontend development
---
```

**Be actionable**: Provide clear guidance, not just information.

```markdown
# Good: Actionable guideline
When testing async components, always:
1. Use `waitFor` for state changes
2. Mock API calls at the network level
3. Test loading and error states

# Bad: Vague advice
Make sure to test your components properly.
```

**Include examples**: Show, don't just tell.

```markdown
## Example: Testing a Form Component

```tsx
test('submits form data', async () => {
  render(<ContactForm />);
  
  await userEvent.type(screen.getByLabelText('Email'), 'test@example.com');
  await userEvent.click(screen.getByRole('button', { name: 'Submit' }));
  
  await waitFor(() => {
    expect(mockSubmit).toHaveBeenCalledWith({ email: 'test@example.com' });
  });
});
```
```

### Companion Files

For complex skills, include additional resources:

```
my-skill/
├── skill.md
├── examples/
│   ├── basic-usage.py
│   └── advanced-patterns.py
├── templates/
│   └── project-structure/
└── reference/
    └── cheatsheet.md
```

Reference these in your skill content:

```markdown
## Getting Started

See `examples/basic-usage.py` for a minimal working example.

For production patterns, refer to `examples/advanced-patterns.py`.
```

## Key Takeaways

1. **Skills are loadable expertise**: They inject domain knowledge into agents on demand, keeping base context lean while enabling deep specialization.

2. **Discovery before loading**: Use `list`, `search`, and `info` operations to find the right skill without consuming context unnecessarily.

3. **Location determines scope**: Workspace skills override user skills, enabling project-specific customization while maintaining personal defaults.

4. **Focus on actionability**: The best skills provide clear guidelines, decision frameworks, and concrete examples—not just reference information.

5. **Companion files extend capabilities**: Skills can include examples, templates, and reference materials that agents access via the `skill_directory` path.

6. **Context is precious**: Load skills judiciously. Each loaded skill consumes tokens that could be used for other context.

---

## Related Concepts

- [Agents](./agents.md) - How agents use skills
- [Bundles](./bundles.md) - Packaging skills with other components
- [Context Management](./context.md) - Managing what agents know
