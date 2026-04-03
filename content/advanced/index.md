---
id: advanced-index
type: reference
title: Advanced Topics
---

# Advanced Topics

Once you've mastered the fundamentals — bundles, tools, skills, providers — Amplifier's deeper machinery opens up. This section is for people who want to *extend* Amplifier, not just use it. Module developers building new capabilities. Contributors working on the kernel itself. Power users pushing multi-repo, multi-agent workflows to their limits.

If you're still getting oriented, start with [Concepts](../concepts/index.md). Everything here assumes you're comfortable with the module system, the session lifecycle, and how bundles compose behavior.

## Who These Topics Are For

| Audience | Start With |
|----------|------------|
| **Module developers** — building tools, hooks, providers | Custom Module Development, Hook Authoring |
| **Workflow architects** — designing multi-step automations | Orchestrator Design, Multi-Repo Workflows |
| **Contributors** — working on Amplifier itself | Shadow Environments, Cross-Repo Testing |
| **Power users** — running Amplifier at scale | Ecosystem Development, Multi-Repo Workflows |

## Section Contents

| Page | Description |
|------|-------------|
| [Custom Bundle](./custom-bundle.md) | Creating custom behavior packages |
| [Custom Recipe](./custom-recipe.md) | Building declarative multi-step workflows |
| [Custom Tool](./custom-tool.md) | Extending Amplifier with new capabilities |
| [MCP Integration](./mcp-integration.md) | Model Context Protocol integration |

## Advanced Topic Map

Beyond the pages above, here's the full landscape of advanced topics and where to find guidance on each.

### Custom Module Development

Every capability in Amplifier is a module that implements `mount()` and follows a protocol contract. Whether you're building a tool, a provider, or a context module, the pattern is the same:

```python
def mount(kernel):
    kernel.register("my-module", handler, protocol="tool/v1")
```

Modules declare their protocol, the kernel validates compliance, and everything else — lifecycle, dependency resolution, hot-reload — is handled automatically. Start with [Custom Tool](./custom-tool.md) for the hands-on walkthrough.

### Hook Authoring

Hooks intercept and transform events at specific points in the kernel lifecycle. They're how you add cross-cutting behavior — logging, guardrails, cost tracking, approval gates — without modifying existing modules.

```python
def mount(kernel):
    kernel.on("before:complete", audit_request)
    kernel.on("after:complete", log_response)
```

> When should I use a hook vs. a tool?

Hooks observe and modify the *flow*. Tools perform *actions*. If you need to inspect every LLM request before it goes out, that's a hook. If you need to run a shell command, that's a tool.

### Orchestrator Design

Orchestrators coordinate multi-agent workflows — deciding which agent handles what, how results combine, and when to escalate. Amplifier's built-in orchestrator handles common patterns, but you can build custom orchestrators for specialized workflows:

- **Fan-out / fan-in** — dispatch to N agents, merge results
- **Pipeline** — chain agents sequentially with context accumulation
- **Competitive** — run multiple agents on the same task, pick the best result
- **Hierarchical** — supervisor agents that delegate to specialist agents

See [Custom Recipe](./custom-recipe.md) for declarative orchestration patterns.

### Shadow Environments

Shadow environments let you test changes to Amplifier modules against real workloads without affecting production behavior. A shadow receives the same inputs as the live system but its outputs are discarded — only logged for comparison.

```yaml
shadow:
  enabled: true
  target: my-experimental-hook
  compare_with: production-hook
  log_divergence: true
```

This is how you validate that a new hook or provider behaves correctly before swapping it in. Shadows are especially useful for provider migrations — run your new provider in shadow mode for a week and compare output quality.

### Multi-Repo Workflows

Real projects span multiple repositories. Amplifier supports cross-repo workflows through workspace-level configuration:

> Set up a multi-repo workspace

```yaml
# ~/.amplifier/workspace.yaml
repositories:
  - path: ~/dev/frontend
    bundles: [react-dev, browser-tester]
  - path: ~/dev/backend
    bundles: [python-dev, api-design]
  - path: ~/dev/shared-libs
    bundles: [library-dev]
```

When you work across repos, Amplifier loads the right bundles for each context. A change in `shared-libs` can trigger reviews in both `frontend` and `backend` through recipe-driven workflows.

### Ecosystem Development

Building for the Amplifier ecosystem means publishing modules, skills, or bundles that others can install. The ecosystem follows a few conventions:

- **Skill publishing** — markdown files following the [Agent Skills spec](https://agentskills.io)
- **Bundle publishing** — YAML packages with declared dependencies
- **Module publishing** — versioned packages with protocol compliance tests
- **Collection publishing** — curated sets of skills, tools, and bundles

> List available ecosystem packages

```
[Tool: bash] amplifier search --type bundle "code review"
  code-review-pro    v2.1.0   Multi-perspective code review
  security-review    v1.3.2   Security-focused code analysis
```

### Cross-Repo Testing Patterns

Testing modules that interact across repositories requires specific patterns:

- **Contract tests** — verify that your module's `mount()` satisfies the protocol
- **Integration tests** — test module behavior against a real kernel instance
- **Shadow tests** — compare your module's output against a known-good baseline
- **Ecosystem tests** — verify your module works with common bundle configurations

```python
# Contract test example
def test_tool_protocol_compliance():
    kernel = TestKernel()
    mount(kernel)
    assert kernel.validates("tool/v1", "my-tool")
```

## Patterns and Anti-Patterns

| Pattern | Do | Don't |
|---------|-----|-------|
| Module scope | One responsibility per module | Monolithic modules doing everything |
| Hook chains | Short, focused hooks | Long chains with hidden dependencies |
| Orchestration | Declarative recipes | Imperative spaghetti in custom code |
| Testing | Contract tests first | Testing only against mocked kernels |
| Configuration | Environment variables for secrets | Hardcoded API keys in config files |

## Complexity Budget

Advanced features add power but also complexity. Apply judiciously:

| Feature | Use When |
|---------|----------|
| Custom modules | Core behavior needs modification |
| Hooks | Cross-cutting concerns (logging, auth, cost) |
| Custom orchestrators | Standard fan-out/pipeline doesn't fit |
| Shadow environments | Validating risky changes pre-deployment |
| Multi-repo | Your project genuinely spans repositories |

If a simpler approach works, use the simpler approach. The advanced machinery exists for cases where the fundamentals aren't enough — not as a default.

## Where to Start

**Building a new tool?** Begin with [Custom Tool](./custom-tool.md) — it walks through the full `mount()` contract.

**Automating workflows?** Start with [Custom Recipe](./custom-recipe.md) for declarative multi-step orchestration.

**Packaging behavior?** See [Custom Bundle](./custom-bundle.md) for creating shareable behavior packages.

**Connecting external tools?** Jump to [MCP Integration](./mcp-integration.md) for Model Context Protocol bridges.

## Next Steps

For day-to-day development workflows, see [Dev Setup](../dev-setup/index.md). For the conceptual foundation behind everything here, revisit [Concepts](../concepts/index.md). For domain-specific knowledge loading, explore [Skills](../skills/index.md).
