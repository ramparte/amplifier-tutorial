---
id: how-its-different
type: quickstart
title: "How Amplifier is Different"
---

# How Amplifier is Different

You might already use AI coding tools like Claude Code, GitHub Copilot, or Cursor. Here's why Amplifier exists and when you might prefer it.

## The Core Difference

**Most AI tools are products. Amplifier is a framework.**

| Aspect | Claude Code / Copilot | Amplifier |
|--------|----------------------|-----------|
| What it is | A finished product | A framework you assemble |
| Provider | Locked to one vendor | Choose from 7+ providers |
| Extensibility | Plugins (limited) | Full module system |
| Workflows | Chat-based | Declarative recipes |
| Transparency | Black box | Event-first observability |
| Multi-agent | Single agent | Native sub-agent spawning |

## Provider Freedom

With most tools, you're locked to one AI provider. With Amplifier:

```bash
# Use Anthropic Claude (recommended)
amplifier provider use anthropic

# Switch to OpenAI
amplifier provider use openai

# Use local models with Ollama (free, private)
amplifier provider use ollama

# Or Azure OpenAI for enterprise compliance
amplifier provider use azure-openai
```

Same tools, same workflows, different brain. You can even compare providers:

```bash
amplifier run "Explain this code" --provider anthropic
amplifier run "Explain this code" --provider openai
```

## Real Extensibility

Copilot has plugins. Amplifier has a **module system** inspired by Linux.

**Tools** - Capabilities the AI can use:
```bash
# See loaded tools
amplifier run "/tools"

# Tools are modules - you can add, remove, or build your own
```

**Hooks** - Observe or modify behavior:
```yaml
# Add approval gates for dangerous commands
hooks:
  - module: hooks-approval
    config:
      require_approval_for: ["bash", "write_file"]
```

**Providers** - Swap the AI backend without changing anything else

**Orchestrators** - Control how the AI executes (streaming, events, basic loop)

## Declarative Workflows (Recipes)

Most tools work through ad-hoc chat. Amplifier supports **recipes** - repeatable, multi-step workflows:

```yaml
# recipes/code-review.yaml
name: code-review
steps:
  - id: analyze
    agent: foundation:zen-architect
    instruction: "Analyze {{file_path}} for design issues"
    
  - id: security
    agent: foundation:security-guardian
    instruction: "Check for vulnerabilities in {{file_path}}"
    
  - id: report
    agent: foundation:modular-builder
    instruction: "Create a review report from {{analyze.result}} and {{security.result}}"
```

Run it:
```bash
amplifier recipes execute recipes/code-review.yaml --context '{"file_path": "src/auth.py"}'
```

Recipes can:
- Run multiple agents in sequence or parallel
- Pause for human approval at critical points
- Resume after interruption
- Pass context between steps

## Transparency

Amplifier logs everything as structured events:

```bash
# Every session creates an events.jsonl file
ls ~/.amplifier/sessions/

# You can inspect exactly what happened
cat ~/.amplifier/sessions/[session-id]/events.jsonl | jq '.event_type'
```

Event types include `session:start`, `turn:start`, `tool:call`, `provider:request`, and more. Nothing is hidden.

## Multi-Agent by Design

Amplifier supports **sub-agents** natively. The main AI can spawn specialists:

```bash
amplifier run "Debug the authentication issue in src/auth.py"
```

Behind the scenes:
1. Main agent recognizes this is a debugging task
2. Spawns `foundation:bug-hunter` sub-agent with full context
3. Bug-hunter investigates, reports back
4. Main agent synthesizes and presents results

You can define your own agents:

```yaml
# agents/my-specialist.yaml
meta:
  name: my-specialist
  description: "Expert at my specific domain"
  
instructions: |
  You are an expert at [my domain].
  When asked about [topic], always [approach].
```

## When to Use What

**Use Claude Code / Copilot when:**
- You want something that "just works"
- You don't need customization
- You're okay with one provider
- Simple chat-based interaction is enough

**Use Amplifier when:**
- You want control over the AI provider
- You need custom tools or workflows
- You want transparent, observable operations
- You're building something on top of AI
- You need multi-agent orchestration
- Enterprise requirements (Azure, compliance)

## The Trade-off

Amplifier requires more setup than "install extension and go." You're assembling a framework, not using a product.

But that assembly gives you power:
- Swap providers when pricing changes
- Build workflows that match your process
- Add tools your team needs
- Observe and audit everything

## Next

Ready to install? Let's get Amplifier running.

→ [Installation](installation.md)
