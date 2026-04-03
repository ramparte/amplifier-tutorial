---
id: provider-community
type: provider
title: "Community Providers"
---

# Community Providers

The official providers cover the major model vendors — Anthropic, OpenAI, Google, Azure, Ollama. But the LLM landscape is vast. Community-contributed providers extend Amplifier to dozens of additional services, each with its own strengths: AWS-native enterprise integration, search-augmented generation, unified proxy layers, high-throughput self-hosted inference, and more. These providers follow the same `complete(request) → response` contract as the official ones. Install them as modules and they slot into the routing matrix like any other provider.

## Overview

Community providers are maintained outside the core Amplifier repository. They're versioned independently, installed via the `amplifier install` command, and configured in your bundle YAML just like official providers. Because every provider implements the same internal protocol, Amplifier doesn't distinguish between official and community — a response from AWS Bedrock looks exactly like a response from Claude to the rest of the system.

> List installed providers

```
[Tool: bash] amplifier providers list
PROVIDER        TYPE        STATUS
anthropic       official    configured
openai          official    configured
bedrock         community   installed
litellm         community   installed
```

Community providers cover two broad categories:

- **Cloud inference services** — Alternative APIs for running models in the cloud (Bedrock, Perplexity, Fireworks)
- **Self-hosted solutions** — Run models on your own infrastructure (vLLM, plus Ollama which is official)

## AWS Bedrock

Enterprise-grade access to multiple model families through AWS. If your organization runs on AWS, Bedrock lets you use Claude, Llama, Mistral, and other models through your existing AWS account with IAM-based authentication, VPC endpoints, and AWS billing.

> Install and configure AWS Bedrock

```
[Tool: bash] amplifier install provider-bedrock
```

```yaml
provider:
  name: bedrock
  region: us-east-1
  model: anthropic.claude-sonnet-4-20250514-v2:0
```

Bedrock uses your AWS credentials automatically — `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` from your environment, or an IAM role if you're running on EC2/ECS. No separate API key needed.

**Best for:** Teams already on AWS who need compliance controls, IAM-based access, and consolidated billing. Supports Claude, Llama, Mistral, and Cohere models through a single AWS integration.

## Perplexity

Search-augmented generation — Perplexity models have built-in web search, so they can answer questions about current events, recent documentation, and topics beyond their training data. Useful for research-heavy workflows where you need up-to-date information without manually searching.

> Install and configure Perplexity

```
[Tool: bash] amplifier install provider-perplexity
```

```yaml
provider:
  name: perplexity
  api_key: ${PERPLEXITY_API_KEY}
  model: sonar-pro
```

```
[Tool: bash] export PERPLEXITY_API_KEY="pplx-..."
```

**Best for:** Research tasks, answering questions about recent events or APIs, workflows that benefit from live web context. Pairs well with Claude for coding — route research to Perplexity and implementation to Anthropic.

## GitHub Copilot

Use your existing GitHub Copilot subscription as an Amplifier provider. If your team already pays for Copilot Business or Enterprise, this avoids setting up separate API keys and billing for another model provider.

> Install and configure GitHub Copilot

```
[Tool: bash] amplifier install provider-copilot
```

```yaml
provider:
  name: copilot
  auth: github
  model: gpt-4o
```

Authentication flows through your GitHub CLI login — run `gh auth login` if you haven't already. The provider uses your Copilot entitlement to access models.

**Best for:** Teams with existing Copilot Enterprise subscriptions who want to consolidate model access. No additional API costs beyond your Copilot license.

## LiteLLM

A unified proxy that sits between Amplifier and over 100 model providers. Instead of configuring each provider individually, you configure LiteLLM once and it handles the translation layer for OpenAI, Anthropic, Azure, Bedrock, Vertex AI, Hugging Face, Replicate, Together AI, and dozens more. If you need to switch between providers frequently or access niche model services, LiteLLM eliminates the integration work.

> Install and configure LiteLLM

```
[Tool: bash] amplifier install provider-litellm
```

```yaml
provider:
  name: litellm
  base_url: http://localhost:4000
  model: anthropic/claude-sonnet-4-20250514
```

LiteLLM runs as a local proxy server. Start it separately:

```
[Tool: bash] pip install litellm
[Tool: bash] litellm --model anthropic/claude-sonnet-4-20250514 --port 4000
```

**Best for:** Teams that need access to many different model providers through a single interface, organizations evaluating multiple vendors, or setups where a central proxy simplifies key management and access control.

## vLLM

High-throughput inference serving for self-hosted models. Where Ollama prioritizes ease of use for single-user local development, vLLM is built for production workloads — continuous batching, PagedAttention for efficient memory use, and tensor parallelism across multiple GPUs. If you're serving models to a team or running high-volume inference, vLLM is the serious option.

> Install and configure vLLM

```
[Tool: bash] amplifier install provider-vllm
```

```yaml
provider:
  name: vllm
  base_url: http://gpu-server:8000
  model: meta-llama/Meta-Llama-3.1-70B-Instruct
```

Start the vLLM server on your GPU machine:

```
[Tool: bash] pip install vllm
[Tool: bash] python -m vllm.entrypoints.openai.api_server \
  --model meta-llama/Meta-Llama-3.1-70B-Instruct \
  --port 8000
```

**Best for:** Teams self-hosting models for multiple developers, production inference workloads, environments where you need Ollama-level privacy with much higher throughput and GPU utilization.

## Fireworks

Fast inference API with competitive pricing. Fireworks optimizes model serving for low latency — they host popular open-source models on custom infrastructure tuned for speed. If you want cloud-hosted open models without managing your own GPU servers, Fireworks fills that gap.

> Install and configure Fireworks

```
[Tool: bash] amplifier install provider-fireworks
```

```yaml
provider:
  name: fireworks
  api_key: ${FIREWORKS_API_KEY}
  model: accounts/fireworks/models/llama-v3p1-70b-instruct
```

```
[Tool: bash] export FIREWORKS_API_KEY="fw_..."
```

**Best for:** Fast inference on open-source models without self-hosting, cost-sensitive workloads where frontier model quality isn't required, teams that want cloud convenience with open model flexibility.

## Mixing Community Providers into Routing

Community providers participate in the routing matrix like any official provider. A team might combine enterprise Bedrock for compliance-sensitive coding with Perplexity for research:

```yaml
providers:
  bedrock:
    region: us-east-1
  perplexity:
    api_key: ${PERPLEXITY_API_KEY}
  ollama:
    base_url: http://localhost:11434

routing:
  coding: bedrock/anthropic.claude-sonnet-4-20250514-v2:0
  research: perplexity/sonar-pro
  fast: ollama/mistral
  reasoning: bedrock/anthropic.claude-sonnet-4-20250514-v2:0
```

The routing matrix doesn't care whether a provider is official or community-contributed. It just dispatches to whatever you've configured.

## Finding More Providers

The community provider ecosystem grows as people contribute new integrations. To discover what's available:

> Search for available community providers

```
[Tool: bash] amplifier search providers
```

You can also check the Amplifier community registry for the latest additions. Any service that exposes an LLM API can be wrapped in a provider module — the contract is simple enough that writing a new one takes an afternoon.

### Writing Your Own Provider

If the provider you need doesn't exist, the `complete(request) → response` contract is straightforward to implement. See [Custom Tool](../advanced/custom-tool.md) for the module pattern — providers follow the same structure. The core requirement: accept Amplifier's normalized request format, call your model's API, and return the response in Amplifier's normalized format.

## Tips

- **Start official, add community as needed.** The official providers cover most use cases. Add community providers when you hit a specific gap — enterprise AWS requirements, search augmentation, self-hosted throughput.
- **Check version compatibility.** Community providers version independently from Amplifier. After upgrading Amplifier, verify your community providers still work.
- **LiteLLM as an escape hatch.** If you need a provider that doesn't have a dedicated module, LiteLLM probably supports it through its proxy. It's the "universal adapter" of the provider ecosystem.
- **vLLM over Ollama for teams.** One developer? Ollama is perfect. Serving models to a team of ten? vLLM's batching and throughput optimization will save you GPU hours.
- **Perplexity for research routing.** Route your research role to Perplexity and let its built-in search handle the web context. Keep your coding role on Claude or GPT where code quality is strongest.
- **Test before committing.** Community providers vary in maturity. Test with a simple prompt before wiring one into your production routing matrix.

## Next Steps

- See the [Provider Index](./index.md) for the full routing matrix configuration and provider comparison
- Explore the official providers: [Anthropic](./anthropic.md), [OpenAI](./openai.md), [Gemini](./gemini.md), [Azure](./azure.md), [Ollama](./ollama.md)
- Learn about [Routing](../concepts/routing.md) to understand how Amplifier dispatches to different providers automatically
