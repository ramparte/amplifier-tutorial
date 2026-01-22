---
id: advanced-index
type: section-index
title: Advanced Topics
---

# Advanced Topics

Once you've mastered the fundamentals, these advanced topics unlock Amplifier's full power. From multi-agent orchestration to custom kernel modules, this section covers expert-level capabilities. Dive deep into sophisticated patterns for building production-grade AI applications.

## Section Contents

| Page | Description |
|------|-------------|
| [Recipes](./recipes.md) | Declarative multi-step workflows |
| [Multi-Agent Patterns](./multi-agent.md) | Orchestrating agent collaboration |
| [Custom Modules](./custom-modules.md) | Extending the kernel |
| [Provider Configuration](./providers.md) | Advanced LLM setup and routing |
| [Session Management](./session-management.md) | Persistence, resume, and repair |
| [Performance Tuning](./performance.md) | Optimization strategies |
| [Security Hardening](./security.md) | Production security practices |
| [Debugging Deep Dive](./debugging.md) | Advanced troubleshooting |

## Quick Tips

- **Recipes for repeatability** - Codify multi-step workflows as recipes
- **Agent specialization** - Create focused agents rather than general-purpose
- **Module contracts** - Follow kernel protocols for reliable extensions
- **Session hygiene** - Clean up and repair sessions proactively
- **Monitor everything** - Observability is essential at scale

## Advanced Patterns

### Recipe-Driven Workflows
```yaml
name: code-review
steps:
  - agent: zen-architect
    mode: REVIEW
    input: "{{file_path}}"
  - agent: security-guardian
    input: "{{previous_output}}"
  - agent: result-validator
    criteria: "{{acceptance_criteria}}"
```

### Multi-Agent Collaboration
```
┌─────────────┐
│ Orchestrator│
└──────┬──────┘
       │
  ┌────┴────┐
  │ Spawn   │
  ▼         ▼
┌───┐     ┌───┐
│ A │     │ B │  Parallel execution
└─┬─┘     └─┬─┘
  │         │
  └────┬────┘
       │
  ┌────▼────┐
  │ Combine │
  └─────────┘
```

## Where to Start

**Automating workflows?** Begin with [Recipes](./recipes.md) for declarative orchestration.

**Building agent systems?** See [Multi-Agent Patterns](./multi-agent.md) for coordination strategies.

**Extending Amplifier?** Start with [Custom Modules](./custom-modules.md) for kernel extension.

## Complexity Budget

Advanced features add power but also complexity. Apply judiciously:

| Feature | Use When |
|---------|----------|
| Recipes | Repeatable multi-step workflows |
| Multi-agent | Tasks benefit from specialization |
| Custom modules | Core behavior needs modification |
| Session management | Long-running or resumable work |

## Next Steps

For day-to-day development workflows, see [Dev Setup](../dev-setup/index.md). For foundational concepts, revisit [Concepts](../concepts/index.md).
