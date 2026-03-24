# Agents Concept - Creating Custom Agents Supplement

This content is injected into the agents concept tutorial page.

## Creating Custom Agents

Agents are defined in bundle YAML files. Here's how to create your own.

### Agent Definition Format

In your bundle's `bundle.md` frontmatter or a behavior YAML:

```yaml
agents:
  my-custom-agent:
    description: "Specializes in database optimization tasks"
    model: "claude-sonnet-4-20250514"  # Optional: override default model
    tools:
      - bash
      - filesystem
      - search
    context:
      include:
        - my-bundle:context/database-patterns.md
```

### Minimal Agent Example

```yaml
agents:
  code-reviewer:
    description: "Reviews code for quality and best practices"
```

That's it! The agent inherits tools from the parent bundle.

### Agent with Custom Instructions

Create a markdown file for the agent's system prompt:

```markdown
<!-- context/agents/code-reviewer.md -->
# Code Reviewer Agent

You are a code review specialist. Focus on:
- Code clarity and readability
- Potential bugs and edge cases
- Performance implications
- Security concerns

Be constructive and specific in feedback.
```

Then reference it:

```yaml
agents:
  code-reviewer:
    description: "Reviews code for quality and best practices"
    context:
      include:
        - my-bundle:context/agents/code-reviewer.md
```

### Agent vs Bundle

| Aspect | Agent | Bundle |
|--------|-------|--------|
| **Is** | A bundle spawned as sub-session | Configuration package |
| **Purpose** | Handle specific tasks | Compose capabilities |
| **Invoked by** | task tool | amplifier CLI or includes |
| **Lifecycle** | Temporary (task duration) | Persistent (session) |

### How Agents Work Internally

```
User: "Review this code for security issues"
         │
         ▼
   ┌─────────────────┐
   │   Main Session  │
   │  (your bundle)  │
   └────────┬────────┘
            │ task tool invoked
            ▼
   ┌─────────────────┐
   │  Agent Session  │  ← New AmplifierSession created
   │ (agent bundle)  │  ← Inherits + overrides from agent config
   └────────┬────────┘
            │ agent completes work
            ▼
   ┌─────────────────┐
   │   Main Session  │  ← Receives result summary
   └─────────────────┘
```

### Try Creating an Agent

1. Add to your bundle.md frontmatter:

```yaml
agents:
  summarizer:
    description: "Summarizes long documents concisely"
```

2. Test it:

```
> Use the summarizer agent to summarize README.md
```
