---
id: provider-anthropic
type: provider
title: "Anthropic Provider"
---

# Anthropic Provider

Anthropic's Claude is the default provider for Amplifier — and for good reason. Claude models consistently produce clean, well-structured code, handle long contexts gracefully, and excel at the kind of multi-step reasoning that real development work demands. If you're new to Amplifier, start here. Most teams never need to look further.

This page covers setup, model selection, and configuration so you can get Claude working in your Amplifier environment quickly.

## Overview

The Anthropic provider connects Amplifier to Claude models through the Anthropic API. It supports the full range of Amplifier capabilities: streaming responses, tool use, extended thinking, and multi-turn conversations. Because Amplifier was designed with Claude as the primary model, integration is deep — features like extended thinking and reasoning traces map directly to Anthropic's API without translation layers.

## Setup

You need one thing: an API key from [console.anthropic.com](https://console.anthropic.com).

There are two ways to provide it.

**Option 1: Environment variable (recommended for personal use)**

> Set your Anthropic API key

```
[Tool: bash] export ANTHROPIC_API_KEY="sk-ant-api03-..."
```

Add it to your shell profile (`~/.bashrc`, `~/.zshrc`) so it persists across sessions.

**Option 2: Keys file (recommended for teams)**

Amplifier checks `~/.amplifier/keys.env` for credentials on startup:

```
[Tool: bash] cat ~/.amplifier/keys.env
ANTHROPIC_API_KEY=sk-ant-api03-...
```

The keys file keeps secrets out of your shell history and lets you manage multiple provider keys in one place. Either approach works — pick the one that fits your workflow.

> Verify the key is set

```
[Tool: bash] echo $ANTHROPIC_API_KEY | head -c 12
sk-ant-api03
```

If you see the prefix, you're good.

## Configuration

Provider configuration lives in your bundle's YAML file. Here's the minimal setup:

```yaml
# amplifier.yaml
provider:
  name: anthropic
  model: claude-sonnet-4-20250514
```

The `api_key` is pulled from your environment automatically. You can also set it explicitly:

```yaml
provider:
  name: anthropic
  api_key: ${ANTHROPIC_API_KEY}
  model: claude-sonnet-4-20250514
  temperature: 0.3
  max_tokens: 8192
```

### Configuration Parameters

| Parameter | Default | Description |
|-----------|---------|-------------|
| **name** | — | Always `anthropic` |
| **model** | `claude-sonnet-4-20250514` | Which Claude model to use |
| **temperature** | `1.0` | Randomness (0.0 = deterministic, 1.0 = creative) |
| **max_tokens** | `8192` | Maximum tokens in the response |
| **api_key** | from env | Explicit key override |

For coding tasks, a temperature of 0.0–0.3 keeps output focused and consistent. For creative work — brainstorming, writing, design exploration — let it run higher.

### Extended Thinking

Claude supports extended thinking, where the model reasons through a problem step by step before producing its answer. This is especially valuable for complex debugging, architectural decisions, and multi-file refactors.

```yaml
provider:
  name: anthropic
  model: claude-sonnet-4-20250514
  temperature: 1.0
  extended_thinking:
    enabled: true
    budget_tokens: 10000
```

When extended thinking is enabled, temperature must be set to `1.0` — the API requires it. The `budget_tokens` parameter caps how many tokens the model can spend reasoning before it commits to an answer. Higher budgets produce deeper analysis but cost more and take longer.

> Solve this concurrency bug — I need you to think carefully

```
[Tool: mode] set reasoning

Reasoning mode activated. Extended thinking enabled with 10K token budget.
Claude will reason step-by-step before responding.
```

You don't usually configure extended thinking manually in YAML — Amplifier's mode system activates it when you need deep reasoning.

## Models

Anthropic offers three model tiers, each optimized for different workloads:

### Claude Opus — Maximum Capability

```yaml
model: claude-opus-4-20250514
```

The most capable Claude model. Opus handles the hardest problems: complex multi-file refactors, deep architectural analysis, nuanced code review, and tasks that require holding large amounts of context simultaneously. It's slower and more expensive than Sonnet, so use it when the problem genuinely demands it.

**Best for:** Hard debugging, large-scale refactoring, architecture design, security audits.

### Claude Sonnet — The Sweet Spot

```yaml
model: claude-sonnet-4-20250514
```

The recommended default. Sonnet delivers strong coding performance at a fraction of Opus's cost and latency. For the vast majority of development tasks — writing features, fixing bugs, generating tests, explaining code — Sonnet is the right choice.

**Best for:** Day-to-day coding, feature implementation, test generation, code explanation.

### Claude Haiku — Speed and Efficiency

```yaml
model: claude-haiku-3-5-20241022
```

The fastest and cheapest Claude model. Haiku excels at quick utility tasks: parsing, classification, simple edits, bulk file operations, and anything where latency matters more than depth. In multi-provider routing, Haiku often fills the "fast" role.

**Best for:** Quick edits, classification, parsing, high-volume tasks, cost-sensitive workloads.

### Model Selection in Routing

You can assign different Claude models to different roles in the routing matrix:

```yaml
providers:
  anthropic:
    api_key: ${ANTHROPIC_API_KEY}

routing:
  coding: anthropic/claude-sonnet-4-20250514
  fast: anthropic/claude-haiku-3-5-20241022
  reasoning: anthropic/claude-opus-4-20250514
  research: anthropic/claude-sonnet-4-20250514
```

This gives you Opus-grade reasoning when you need it, Haiku speed for quick tasks, and Sonnet for everything in between — all without manually switching models.

## Tips

- **Start with Sonnet.** It handles 90% of coding tasks well. Only reach for Opus when you hit a problem Sonnet can't crack.
- **Use extended thinking for hard problems.** If you're debugging a subtle race condition or planning a complex refactor, reasoning mode is worth the extra tokens.
- **Lower temperature for code.** Set `temperature: 0.0` or `0.1` when you want deterministic, reproducible output — especially for generated tests or structured data.
- **Let routing pick the model.** Instead of manually choosing models, configure the routing matrix and let Amplifier dispatch to the right tier automatically.
- **Watch your max_tokens.** If responses are getting cut off, bump `max_tokens`. If you're paying too much for simple queries, lower it.
- **Keys file for teams.** If you're sharing a machine or using shared configuration, `~/.amplifier/keys.env` keeps API keys out of your YAML files and shell history.

## Next Steps

- See the [Provider Index](./index.md) for multi-provider routing and how to combine Claude with other models
- Learn about [Modes](../concepts/modes.md) to understand how Amplifier activates extended thinking automatically
- Explore the [OpenAI Provider](./openai.md) or [Gemini Provider](./gemini.md) if you need additional model capabilities
