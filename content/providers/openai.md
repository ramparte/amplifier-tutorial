---
id: provider-openai
type: provider
title: "OpenAI Provider"
---

# OpenAI Provider

OpenAI's GPT models bring a different set of strengths to Amplifier. GPT-4o excels at vision tasks — analyzing screenshots, reading diagrams, understanding UI mockups. The o-series reasoning models take a distinct approach to complex problem-solving. And GPT-4o-mini delivers fast, inexpensive responses for high-volume utility work. Whether you use OpenAI as your primary provider or pair it with Claude in a multi-provider setup, the integration is seamless.

## Overview

The OpenAI provider connects Amplifier to GPT models through the OpenAI API. It supports streaming, tool use (function calling), vision inputs, and multi-turn conversations. OpenAI models slot into Amplifier's routing matrix like any other provider — the kernel doesn't care which vendor answered, only that the response follows the normalized format.

## Setup

Get an API key from [platform.openai.com](https://platform.openai.com).

> Set your OpenAI API key

```
[Tool: bash] export OPENAI_API_KEY="sk-proj-..."
```

Add it to `~/.bashrc` or `~/.zshrc` for persistence, or use Amplifier's keys file:

```
[Tool: bash] cat ~/.amplifier/keys.env
OPENAI_API_KEY=sk-proj-...
```

> Verify the connection

```
> What model are you running on?

I'm running through the OpenAI provider with GPT-4o.
Your Amplifier session is configured to route through openai/gpt-4o.
```

If you get a response, the key is working.

## Configuration

Basic configuration in your bundle YAML:

```yaml
# amplifier.yaml
provider:
  name: openai
  model: gpt-4o
```

Full configuration with all parameters:

```yaml
provider:
  name: openai
  api_key: ${OPENAI_API_KEY}
  model: gpt-4o
  temperature: 0.2
  max_tokens: 4096
  top_p: 1.0
```

### Configuration Parameters

| Parameter | Default | Description |
|-----------|---------|-------------|
| **name** | — | Always `openai` |
| **model** | `gpt-4o` | Which GPT model to use |
| **temperature** | `1.0` | Randomness (0.0–2.0, lower = more focused) |
| **max_tokens** | `4096` | Maximum tokens in the response |
| **top_p** | `1.0` | Nucleus sampling (alternative to temperature) |
| **api_key** | from env | Explicit key override |

Use either `temperature` or `top_p` to control randomness — not both. For coding, `temperature: 0.0–0.2` keeps output tight.

## Models

OpenAI offers several model families, each with distinct characteristics.

### GPT-4o — The Generalist

```yaml
model: gpt-4o
```

OpenAI's flagship multimodal model. GPT-4o handles text and images natively, making it the go-to choice for vision tasks in Amplifier. It's fast, capable, and cost-effective for its tier. Strong at coding, explanation, and general-purpose tasks.

**Best for:** Vision tasks, UI analysis, general coding, multi-modal workflows.

### GPT-4o-mini — Fast and Cheap

```yaml
model: gpt-4o-mini
```

A smaller, faster version of GPT-4o. Significantly cheaper per token with lower latency. It handles straightforward tasks well — parsing, classification, simple code edits, formatting. In multi-provider routing, this is a natural fit for the "fast" role.

**Best for:** Quick tasks, classification, parsing, high-volume operations, cost-sensitive workloads.

### GPT-4 Turbo — Legacy Workhorse

```yaml
model: gpt-4-turbo
```

The previous generation flagship. Still capable, but GPT-4o is generally preferred — it's faster, cheaper, and matches or exceeds GPT-4 Turbo on most benchmarks. Use GPT-4 Turbo if you have specific compatibility requirements.

**Best for:** Backward compatibility, specific fine-tuned workflows.

### o1 and o3 — Reasoning Models

```yaml
model: o3
```

OpenAI's reasoning models take a fundamentally different approach. Instead of generating a response in one pass, they spend time "thinking" — exploring solution paths, checking their work, and backtracking when necessary. This makes them exceptionally strong at math, logic, and complex multi-step reasoning.

```yaml
model: o1
```

The o-series models have some differences from standard GPT models:

- **No temperature control.** The models manage their own sampling internally.
- **No streaming.** Responses arrive all at once after reasoning completes.
- **Higher latency.** Thinking takes time — expect seconds to minutes for hard problems.
- **Higher cost.** Reasoning tokens are billed alongside completion tokens.

```yaml
# Reasoning model configuration
provider:
  name: openai
  model: o3
  max_tokens: 16384
```

**Best for:** Complex math, formal logic, multi-step debugging, problems that benefit from deliberate reasoning.

### Vision Capabilities

GPT-4o processes images natively. In Amplifier, this means you can pass screenshots, diagrams, or mockups directly into the conversation:

> Look at this screenshot and tell me what's wrong with the layout

```
[Tool: bash] cat screenshot.png | base64 > /tmp/img.b64

The header nav is overlapping the hero section. The z-index on
.navbar is set to 10 but the hero has z-index: 100, which puts
it above the nav. Also, the flex container on .hero-content is
missing align-items, so the text isn't vertically centered.
```

Vision is especially useful for:
- Reviewing UI implementations against mockups
- Reading error messages from screenshots
- Analyzing architecture diagrams
- Understanding whiteboard photos

### Function Calling

OpenAI models support function calling natively — this is how Amplifier's tool system works under the hood. When the model decides it needs to read a file, run a command, or search code, it generates a structured function call that Amplifier executes. You don't configure this directly; it's handled by the provider layer automatically.

The function calling protocol maps cleanly to Amplifier's tool definitions, so all tools work with OpenAI models out of the box.

## Multi-Provider Routing with OpenAI

OpenAI models pair well with Claude in a routing matrix. A common pattern uses Claude for core coding and OpenAI for vision and fast tasks:

```yaml
providers:
  anthropic:
    api_key: ${ANTHROPIC_API_KEY}
  openai:
    api_key: ${OPENAI_API_KEY}

routing:
  coding: anthropic/claude-sonnet-4-20250514
  fast: openai/gpt-4o-mini
  vision: openai/gpt-4o
  creative: openai/gpt-4o
  reasoning: anthropic/claude-sonnet-4-20250514
```

This gives you the best of both worlds — Claude's coding strength for implementation work, GPT-4o's vision for UI tasks, and GPT-4o-mini's speed for quick operations.

## Tips

- **Use GPT-4o for vision.** If your workflow involves screenshots, diagrams, or UI review, GPT-4o is the strongest choice. Route vision tasks to it even if Claude is your primary provider.
- **GPT-4o-mini for bulk work.** Processing many files, classifying issues, or generating boilerplate? Mini is fast and cheap — don't spend GPT-4o tokens on simple tasks.
- **Reasoning models need patience.** o1 and o3 are powerful but slow. Use them for genuinely hard problems, not routine coding.
- **No temperature on o-series.** Don't set temperature when using o1 or o3 — the API will reject it. These models manage randomness internally.
- **Bump max_tokens for long responses.** GPT models default to shorter completions than Claude. If responses are getting truncated, increase `max_tokens` to 8192 or higher.
- **Check rate limits.** OpenAI enforces per-minute token limits that vary by tier. If you're seeing 429 errors, you've hit the ceiling — wait or upgrade your plan.

## Next Steps

- See the [Provider Index](./index.md) for the full routing matrix configuration
- Compare with the [Anthropic Provider](./anthropic.md) to understand the tradeoffs between Claude and GPT
- Explore the [Gemini Provider](./gemini.md) for Google's multimodal models
