---
id: how-its-different
type: quickstart
title: "How It's Different"
---

# How Amplifier is Different

Amplifier takes a fundamentally different approach to building AI-powered applications. Rather than providing a monolithic framework or a simple API wrapper, Amplifier offers a modular kernel that you compose into exactly what you need.

## Comparison Overview

Most AI development tools fall into one of two categories:

| Approach | Examples | Trade-offs |
|----------|----------|------------|
| **Monolithic Frameworks** | Full-stack AI platforms | Feature-rich but opinionated, hard to customize |
| **API Wrappers** | Thin client libraries | Flexible but low-level, requires building everything |
| **Amplifier** | Modular kernel + bundles | Compose what you need, extend without forking |

### What Makes Amplifier Different

1. **Kernel + Module Architecture**: A minimal core with pluggable capabilities
2. **Composition Over Configuration**: Build your stack by composing bundles
3. **First-Class Multi-Agent Support**: Agents that collaborate, not just respond
4. **Open Source Philosophy**: Inspect, modify, and contribute to every layer

## Modular Architecture

Amplifier's architecture follows the "bricks and studs" philosophy—small, well-defined modules that snap together through standard interfaces.

### The Kernel

The kernel is intentionally minimal. It provides:

- **Session lifecycle management**: Start, run, persist, resume
- **Event system**: Hooks and observers for extensibility
- **Module protocol**: Standard contracts for all module types
- **Provider abstraction**: Swap LLM backends without code changes

The kernel does NOT include:
- Specific tools or capabilities
- Business logic
- Opinionated workflows

### Module Types

Amplifier defines clear module contracts:

```
Provider     → LLM backends (Anthropic, OpenAI, Azure, etc.)
Tool         → Capabilities the agent can use
Hook         → Intercept and modify agent behavior
Observer     → React to events without modification
Context      → Inject information into agent sessions
```

### Bundles: Composable Capability Packs

Bundles group related modules into reusable packages:

```yaml
# Example: A bundle for web development
name: web-dev-bundle
modules:
  tools:
    - browser-automation
    - http-client
    - html-parser
  context:
    - web-standards
    - accessibility-guidelines
```

You compose your Amplifier instance by selecting bundles:

```yaml
# Your application's bundle composition
extends:
  - foundation          # Core capabilities
  - web-dev-bundle      # Web development tools
  - your-custom-bundle  # Your domain-specific additions
```

## Extensibility

Traditional frameworks force a choice: use what they provide, or fork and maintain your own version. Amplifier offers a third path.

### Extend Without Forking

Every extension point in Amplifier is designed for external modification:

| Extension Point | What You Can Do |
|-----------------|-----------------|
| **Providers** | Add new LLM backends or modify existing ones |
| **Tools** | Create custom tools with full type safety |
| **Hooks** | Transform requests/responses in the agent loop |
| **Observers** | Add logging, metrics, or side effects |
| **Context** | Inject domain knowledge and instructions |

### Hook System Example

Hooks let you intercept the agent loop at defined points:

```python
class ContentFilterHook(Hook):
    """Filter sensitive content before it reaches the LLM."""
    
    async def pre_request(self, messages: list) -> list:
        return [self.filter_pii(m) for m in messages]
    
    async def post_response(self, response: Response) -> Response:
        return self.validate_output(response)
```

### Skills: Loadable Domain Knowledge

Skills are structured knowledge that agents can load on demand:

```python
# Agent loads a skill for specific domain expertise
load_skill("python-standards")
load_skill("security-best-practices")
```

Skills keep specialized knowledge modular and maintainable.

## Multi-Agent

Amplifier treats multi-agent patterns as first-class citizens, not afterthoughts.

### Agent Hierarchy

Agents in Amplifier can spawn sub-agents for specialized tasks:

```
Main Agent
├── Explorer Agent      → Deep codebase reconnaissance
├── Builder Agent       → Implementation from specs
├── Reviewer Agent      → Code quality assessment
└── Test Agent          → Verification and validation
```

### Delegation Patterns

The `task` tool enables clean agent delegation:

```python
# Main agent delegates to a specialist
result = await task(
    agent="foundation:bug-hunter",
    instruction="Investigate the KeyError in auth.py line 42"
)
```

Each sub-agent:
- Runs in its own session
- Has its own tool access
- Returns a structured result
- Cannot access parent context (isolation by default)

### Parallel Execution

Independent tasks run in parallel:

```python
# These execute concurrently
results = await parallel([
    task(agent="test-runner", instruction="Run unit tests"),
    task(agent="linter", instruction="Check code style"),
    task(agent="security", instruction="Scan for vulnerabilities"),
])
```

### Recipes: Declarative Workflows

For complex multi-step workflows, Amplifier provides recipes:

```yaml
name: code-review-workflow
steps:
  - agent: explorer
    instruction: Map the codebase structure
    
  - agent: reviewer
    instruction: Review changes against standards
    depends_on: [0]
    
  - agent: summarizer
    instruction: Create review summary
    depends_on: [1]
```

## Open Source

Amplifier is fully open source under the MIT license. But open source means more than just "the code is available."

### Inspect Everything

Every layer of Amplifier is transparent:

- **Kernel source**: See exactly how the agent loop works
- **Bundle contents**: Inspect any tool, hook, or context
- **Session logs**: Full event history for debugging
- **Decision traces**: Understand why agents made choices

### Modify Anything

You can override any component:

```yaml
# Override a specific tool from a bundle
modules:
  tools:
    - name: file-writer
      override: my-custom-file-writer
```

### Contribute Back

The modular architecture makes contributions clean:

- Add a new provider? Create a provider module
- Build a useful tool? Package it as a bundle
- Improve the kernel? Submit a focused PR

No need to understand the entire codebase to contribute meaningfully.

### Community-Driven Evolution

Amplifier's design supports:

- **Bundle ecosystem**: Share and discover community bundles
- **Skill libraries**: Domain knowledge anyone can contribute
- **Recipe patterns**: Proven workflows for common tasks

## Summary

| Aspect | Traditional Approach | Amplifier Approach |
|--------|---------------------|-------------------|
| **Architecture** | Monolithic or minimal | Modular kernel + composable bundles |
| **Customization** | Fork or workaround | Extend through standard contracts |
| **Multi-Agent** | Manual orchestration | First-class delegation and recipes |
| **Transparency** | Black box behaviors | Full inspection at every layer |
| **Philosophy** | Feature accumulation | Ruthless simplicity |

Amplifier isn't trying to be everything to everyone. It's designed to be exactly what you need—composed from well-defined pieces that you control.

## Next Steps

- [Installation](./installation.md): Get Amplifier installed and ready
- [First Conversation](./first-conversation.md): Start your first session
- [Concepts](../concepts/index.md): Understand the fundamental building blocks
- [Bundles](../bundles/index.md): Learn to compose your own stack
