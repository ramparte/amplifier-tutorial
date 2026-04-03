---
id: provider-ollama
type: provider
title: "Ollama Provider"
---

# Ollama Provider

No API keys. No cloud. No data leaving your machine. Ollama runs open-source LLMs directly on your hardware — your laptop, your workstation, your air-gapped server in the basement. The tradeoff is obvious: local models are smaller and slower than cloud-hosted frontier models. But for privacy-sensitive work, offline development, or just experimenting without burning API credits, nothing beats a model that runs entirely under your control.

## Overview

The Ollama provider connects Amplifier to models running through [Ollama](https://ollama.ai), a local model server that manages downloading, running, and serving open-source LLMs. Ollama exposes an OpenAI-compatible API on `localhost`, so Amplifier talks to it exactly like it would talk to any cloud provider — except the inference happens on your CPU or GPU.

This is the only provider where your conversation data never touches a network. Not encrypted in transit, not stored on someone else's server — it simply never leaves your machine.

## Setup

### Step 1: Install Ollama

> Install Ollama on your system

```
[Tool: bash] curl -fsSL https://ollama.ai/install.sh | sh
```

On macOS, you can also install via Homebrew:

```
[Tool: bash] brew install ollama
```

On Windows, download the installer from [ollama.ai](https://ollama.ai).

### Step 2: Start the Server

> Start the Ollama server

```
[Tool: bash] ollama serve
Ollama is running on http://127.0.0.1:11434
```

On macOS and Windows, the Ollama desktop app starts the server automatically. On Linux, you may want to run it as a systemd service for persistence.

### Step 3: Pull a Model

Ollama doesn't include models by default — you pull them like Docker images.

> Pull some useful models for development

```
[Tool: bash] ollama pull llama3.1
pulling manifest... done
pulling model... 100% ████████████████████ 4.7 GB
```

```
[Tool: bash] ollama pull codellama
pulling manifest... done
pulling model... 100% ████████████████████ 3.8 GB
```

```
[Tool: bash] ollama pull mistral
pulling manifest... done
pulling model... 100% ████████████████████ 4.1 GB
```

> See what's available locally

```
[Tool: bash] ollama list
NAME              SIZE    MODIFIED
llama3.1:latest   4.7 GB  2 minutes ago
codellama:latest  3.8 GB  1 minute ago
mistral:latest    4.1 GB  30 seconds ago
```

### Step 4: Verify the Connection

> Test that Ollama responds

```
[Tool: bash] curl -s http://localhost:11434/api/tags | python3 -m json.tool | head -10
{
    "models": [
        {
            "name": "llama3.1:latest",
            "size": 4661223424,
            ...
```

If you see your models listed, the server is ready.

## Configuration

### Minimal Configuration

```yaml
# amplifier.yaml
provider:
  name: ollama
  model: llama3.1
```

The `base_url` defaults to `http://localhost:11434` — no need to specify it unless you've changed the port or are running Ollama on another machine.

### Full Configuration

```yaml
provider:
  name: ollama
  base_url: http://localhost:11434
  model: llama3.1
  temperature: 0.2
  max_tokens: 4096
```

### Configuration Parameters

| Parameter | Default | Description |
|-----------|---------|-------------|
| **name** | — | Always `ollama` |
| **model** | — | Model name as shown in `ollama list` |
| **base_url** | `http://localhost:11434` | Ollama server address |
| **temperature** | `0.8` | Randomness (0.0 = deterministic, higher = creative) |
| **max_tokens** | `2048` | Maximum tokens in the response |

### Remote Ollama Server

Running Ollama on a beefy GPU server while developing on a lighter laptop? Point Amplifier at the remote instance:

```yaml
provider:
  name: ollama
  base_url: http://gpu-server.local:11434
  model: llama3.1:70b
```

Just make sure the server is reachable on your network. Ollama binds to localhost by default — set `OLLAMA_HOST=0.0.0.0` on the server to accept remote connections.

## Models

Open-source models vary widely in capability. Here's what works well with Amplifier.

### Llama 3.1 — Best General Purpose

```yaml
model: llama3.1
```

Meta's flagship open model. Llama 3.1 handles general coding, explanation, and conversation well. The 8B parameter version runs on most laptops; the 70B version needs a serious GPU but approaches cloud model quality.

```
[Tool: bash] ollama pull llama3.1        # 8B - runs on most hardware
[Tool: bash] ollama pull llama3.1:70b    # 70B - needs ~40GB VRAM
```

**Best for:** General coding tasks, conversation, explanation, the default choice.

### CodeLlama — Code-Specialized

```yaml
model: codellama
```

Fine-tuned specifically for code generation. CodeLlama understands programming patterns better than general models of the same size. Supports fill-in-the-middle completions natively.

**Best for:** Code generation, completion, code-focused tasks on limited hardware.

### Mistral — Fast and Compact

```yaml
model: mistral
```

Mistral's 7B model punches above its weight. Fast inference, small memory footprint, and solid general performance. A good choice when you need speed over depth.

**Best for:** Quick tasks, limited hardware, fast iteration cycles.

### DeepSeek Coder — Strong at Code

```yaml
model: deepseek-coder-v2
```

A code-specialized model with strong benchmark performance. Handles complex code generation and understanding well for its size class.

**Best for:** Code-heavy workflows where you need better-than-average local code quality.

### Choosing Model Sizes

The `:latest` tag typically pulls the smallest variant. Specify sizes explicitly:

```
[Tool: bash] ollama pull llama3.1:8b     # 8B params, ~5GB, runs on 8GB RAM
[Tool: bash] ollama pull llama3.1:70b    # 70B params, ~40GB, needs serious GPU
```

**Rule of thumb:** Use the largest model your hardware can run at acceptable speed. An 8B model responding in 2 seconds beats a 70B model that takes 30 seconds — for most interactive work.

## A Fully Local Amplifier Environment

Here's the complete setup for running Amplifier with zero cloud dependencies:

> Set up a fully offline Amplifier workspace

```yaml
# amplifier.yaml — fully local configuration
provider:
  name: ollama
  base_url: http://localhost:11434
  model: llama3.1

# No API keys needed anywhere
# No network access required
# All data stays on this machine
```

```
[Tool: bash] ollama serve &
[Tool: bash] ollama pull llama3.1
```

> Verify the local setup works

```
> Write a Python function to parse CSV files

Here's a CSV parser using the standard library...
```

Everything runs locally. The conversation, the model inference, the tool execution — all on your machine. This is ideal for:

- Working on classified or highly sensitive codebases
- Development in air-gapped environments
- Planes, trains, and places without internet
- Experimenting without spending money on API calls

## Tips

- **Start with llama3.1.** It's the best general-purpose local model. Branch out once you know your needs.
- **GPU makes a huge difference.** CPU inference works but is slow — 10-30x slower than GPU. If you have an NVIDIA GPU, Ollama uses it automatically via CUDA.
- **Monitor memory usage.** Models load into RAM (or VRAM). An 8B model needs ~5GB, 70B needs ~40GB. If your system starts swapping, use a smaller model.
- **Expect shorter, simpler responses.** Local models are less capable than cloud frontier models. They'll handle straightforward tasks well but may struggle with complex multi-step reasoning.
- **Use Ollama for privacy, cloud for power.** A common pattern: Ollama for sensitive code, Claude or GPT for everything else. The routing matrix makes this seamless.
- **Keep models updated.** Run `ollama pull <model>` periodically — new quantizations and model versions improve quality without hardware changes.
- **Run the server as a service.** On Linux, set up a systemd unit so Ollama starts on boot. No need to remember `ollama serve` every morning.

## Next Steps

- See the [Provider Index](./index.md) for mixing Ollama with cloud providers in a routing matrix
- Explore [Community Providers](./community.md) for vLLM if you need higher-throughput local serving
- Check the [Anthropic Provider](./anthropic.md) or [OpenAI Provider](./openai.md) for cloud alternatives when you need more capable models
