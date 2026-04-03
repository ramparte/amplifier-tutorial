---
id: providers-index
type: reference
title: Provider Index
---

# Provider Index

Amplifier doesn't lock you into a single AI vendor. Every model interaction flows through a **provider module** — a thin adapter that translates Amplifier's internal request format into whatever a particular LLM API expects. Swap providers, mix providers, or run models locally. The rest of the system never notices.

This page maps out every provider available today, shows you how to configure each one, and explains how multi-provider routing works.

## Section Contents

| Page | Description |
|------|-------------|
| [Anthropic (Claude)](./anthropic.md) | Default provider — Claude models for code and reasoning |
| [OpenAI (GPT)](./openai.md) | GPT-4o, GPT-4 Turbo, and o-series reasoning models |
| [Google Gemini](./gemini.md) | Gemini Pro and Ultra with native multimodal support |
| [Azure OpenAI](./azure.md) | Enterprise-grade OpenAI with compliance and data residency |
| [Ollama (Local)](./ollama.md) | Run models locally — no API keys, no data leaving your machine |
| [Community Providers](./community.md) | Third-party and community-maintained provider modules |

## The Provider Contract

Every provider module implements one function:

```python
complete(request) → response
```

That's the entire contract. The `request` carries the conversation history, model parameters, and tool definitions. The `response` returns the model's output in Amplifier's normalized format. Streaming, tool calls, token counting — all handled inside the provider so nothing upstream needs to care which model answered.

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│  Amplifier   │────▶│   Provider   │────▶│   LLM API    │
│   Kernel     │◀────│   Module     │◀────│  (any vendor) │
└──────────────┘     └──────────────┘     └──────────────┘
```

Because every provider speaks the same internal protocol, switching from Claude to GPT is a config change — not a code change.

## Official Providers

These providers ship with Amplifier and are actively maintained.

### Anthropic (Claude)

The default provider. Claude models excel at code generation, long-context reasoning, and tool use.

> Set up Claude as your provider

```yaml
# amplifier.yaml
provider:
  name: anthropic
  api_key: ${ANTHROPIC_API_KEY}
  model: claude-sonnet-4-20250514
```

```
[Tool: bash] export ANTHROPIC_API_KEY="sk-ant-..."
```

### OpenAI (GPT)

Full support for GPT-4o, GPT-4 Turbo, and o-series reasoning models.

> Configure OpenAI

```yaml
provider:
  name: openai
  api_key: ${OPENAI_API_KEY}
  model: gpt-4o
```

```
[Tool: bash] export OPENAI_API_KEY="sk-..."
```

### Google Gemini

Access to Gemini Pro and Ultra models with native multimodal support.

> Configure Gemini

```yaml
provider:
  name: google
  api_key: ${GOOGLE_API_KEY}
  model: gemini-2.5-pro
```

### Azure OpenAI

For teams that need Azure's compliance, data residency, and enterprise controls.

> Configure Azure OpenAI

```yaml
provider:
  name: azure-openai
  api_key: ${AZURE_OPENAI_API_KEY}
  endpoint: https://your-resource.openai.azure.com
  deployment: gpt-4o-deployment
  api_version: "2024-06-01"
```

Azure requires both an endpoint and a deployment name — the model is tied to the deployment, not specified directly.

### Ollama (Local Models)

Run models entirely on your machine. No API keys, no network calls, no data leaving your laptop.

> Configure Ollama for local inference

```yaml
provider:
  name: ollama
  endpoint: http://localhost:11434
  model: llama3.1:70b
```

```
[Tool: bash] ollama pull llama3.1:70b
[Tool: bash] ollama serve
```

Ollama is ideal for air-gapped environments, sensitive codebases, or just experimenting without burning API credits.

## Community Providers

These providers are maintained by the community. They follow the same `complete(request) → response` contract and install as standard modules.

| Provider | Best For | Install |
|----------|----------|---------|
| **AWS Bedrock** | Enterprise AWS environments, managed scaling | `amplifier install provider-bedrock` |
| **Perplexity** | Research-heavy tasks with built-in web search | `amplifier install provider-perplexity` |
| **GitHub Copilot** | Teams already on Copilot enterprise plans | `amplifier install provider-copilot` |
| **LiteLLM** | Unified proxy for 100+ providers | `amplifier install provider-litellm` |
| **vLLM** | Self-hosted high-throughput inference | `amplifier install provider-vllm` |
| **Fireworks** | Fast inference with competitive pricing | `amplifier install provider-fireworks` |

Community providers are versioned independently. Check each module's docs for supported models and configuration options.

## Multi-Provider Configuration

Here's where it gets interesting. Amplifier's **routing matrix** lets you assign different providers to different roles — so your coding tasks use one model while your research tasks use another.

> Set up a multi-provider routing matrix

```yaml
# amplifier.yaml
providers:
  anthropic:
    api_key: ${ANTHROPIC_API_KEY}
  openai:
    api_key: ${OPENAI_API_KEY}
  ollama:
    endpoint: http://localhost:11434

routing:
  coding: anthropic/claude-sonnet-4-20250514
  fast: openai/gpt-4o-mini
  research: anthropic/claude-sonnet-4-20250514
  reasoning: anthropic/claude-sonnet-4-20250514
  creative: openai/gpt-4o
  security-audit: anthropic/claude-sonnet-4-20250514
  vision: openai/gpt-4o
```

Each role maps to a provider/model pair. When Amplifier dispatches a task, the routing matrix picks the right model automatically. You never manually select — you just work, and the system routes.

> Check which provider handles each role

```
[Tool: mode] operation: current
Active routing matrix: balanced
  coding      → anthropic/claude-sonnet-4-20250514
  fast        → openai/gpt-4o-mini
  research    → anthropic/claude-sonnet-4-20250514
  creative    → openai/gpt-4o
```

## Provider Selection Criteria

Choosing providers is a tradeoff. Here's a framework:

| Criterion | Best Choice | Why |
|-----------|-------------|-----|
| **Cost** | Ollama, GPT-4o-mini | Local = free; mini models are cheap |
| **Speed** | GPT-4o-mini, Fireworks | Optimized for low latency |
| **Code quality** | Claude Sonnet | Strong at structured code generation |
| **Reasoning depth** | Claude Sonnet, o-series | Extended thinking for hard problems |
| **Privacy** | Ollama, vLLM | Data never leaves your infrastructure |
| **Multimodal** | GPT-4o, Gemini Pro | Native image and vision support |
| **Enterprise compliance** | Azure OpenAI, Bedrock | SOC2, HIPAA, data residency |

Most teams start with a single provider and add more as specific needs emerge. The routing matrix makes it painless to experiment — change one line, see if the results improve.

## Quick Setup Checklist

For any provider, setup follows the same pattern:

1. **Get credentials** — API key, endpoint, or local server
2. **Set environment variable** — `export PROVIDER_API_KEY="..."`
3. **Add to config** — provider block in `amplifier.yaml`
4. **Verify** — run a simple prompt and confirm the response

> Verify your provider is working

```
> What model are you?

I'm Claude, made by Anthropic. Your Amplifier session is routing
through the anthropic provider with claude-sonnet-4-20250514.
```

If you see a response, you're connected. If not, check that your API key is set and the endpoint is reachable.

## Next Steps

With providers configured, explore [Routing](../concepts/routing.md) to understand how Amplifier selects models for different tasks. For building your own provider module, see [Custom Tool](../advanced/custom-tool.md) — the module contract is the same pattern.
