---
id: provider-gemini
type: provider
title: "Gemini Provider"
---

# Gemini Provider

Google's Gemini models bring something the other providers can't match: truly massive context windows and native multimodal understanding across text, images, video, and audio. When you need to reason over an entire codebase at once, analyze a video walkthrough of a bug, or generate images from descriptions, Gemini opens doors that other models keep closed.

## Overview

The Gemini provider connects Amplifier to Google's Gemini model family through the Google AI API. It supports streaming, tool use, multimodal inputs, and Amplifier's full routing system. Gemini models process text, images, video, and audio natively — not through translation layers — which means multimodal tasks run faster and produce better results than bolt-on approaches.

## Setup

Get an API key from [ai.google.dev](https://ai.google.dev).

> Set your Google API key

```
[Tool: bash] export GOOGLE_API_KEY="AIza..."
```

Add it to your shell profile or use the keys file:

```
[Tool: bash] cat ~/.amplifier/keys.env
GOOGLE_API_KEY=AIza...
```

> Verify the provider is working

```
> What model are you?

I'm Gemini 2.5 Pro, running through the Google provider in your
Amplifier session.
```

If you see a response, the key is active and the provider is connected.

## Configuration

Basic setup in your bundle YAML:

```yaml
# amplifier.yaml
provider:
  name: google
  model: gemini-2.5-pro
```

Full configuration with all parameters:

```yaml
provider:
  name: google
  api_key: ${GOOGLE_API_KEY}
  model: gemini-2.5-pro
  temperature: 0.3
  max_tokens: 8192
  top_p: 0.95
  top_k: 40
```

### Configuration Parameters

| Parameter | Default | Description |
|-----------|---------|-------------|
| **name** | — | Always `google` |
| **model** | `gemini-2.5-pro` | Which Gemini model to use |
| **temperature** | `1.0` | Randomness (0.0–2.0, lower = more deterministic) |
| **max_tokens** | `8192` | Maximum tokens in the response |
| **top_p** | `0.95` | Nucleus sampling threshold |
| **top_k** | `40` | Top-K sampling (unique to Gemini) |
| **api_key** | from env | Explicit key override |

Gemini supports both `top_p` and `top_k` sampling — `top_k` limits the candidate tokens at each step to the K most likely options. For coding, `temperature: 0.0–0.2` with `top_k: 20` produces tight, consistent output.

## Models

Gemini ships in three tiers, each with different speed-capability-cost tradeoffs.

### Gemini Pro — Deep Reasoning

```yaml
model: gemini-2.5-pro
```

The most capable Gemini model. Gemini Pro handles complex reasoning, large-context analysis, and sophisticated code generation. Its standout feature is the context window — up to 1 million tokens, which means you can feed it entire codebases, lengthy documentation sets, or long conversation histories without truncation.

**Best for:** Large-codebase analysis, long-document reasoning, complex coding tasks, research synthesis.

### Gemini Flash — Speed First

```yaml
model: gemini-2.5-flash
```

Optimized for speed and cost. Flash delivers good-quality responses at significantly lower latency and price than Pro. It still supports multimodal inputs and the large context window, making it a strong choice for tasks where speed matters more than maximum depth.

**Best for:** Quick tasks, real-time workflows, high-volume processing, cost-sensitive operations.

### Gemini Ultra — Maximum Capability

```yaml
model: gemini-ultra
```

The largest and most capable model in the Gemini family. Ultra pushes the boundaries on the hardest tasks — deep mathematical reasoning, complex multi-step problems, and nuanced understanding of difficult domains. Use it when Pro isn't enough and you need every bit of capability available.

**Best for:** Hardest reasoning problems, cutting-edge research, tasks where Pro falls short.

## Long Context Window

This is Gemini's defining advantage. With a 1-million-token context window, you can process amounts of text that would require chunking and summarization with other providers.

> Analyze the entire src/ directory for architectural patterns

```
[Tool: bash] find src/ -name "*.py" -exec cat {} + | wc -c
847293 characters (~200K tokens)
```

```
Gemini can process the entire codebase in a single pass. Here's
what I found:

1. Repository pattern in src/data/ — all DB access goes through
   repository classes, never direct queries
2. Service layer in src/services/ — business logic separated
   from HTTP handling
3. Event-driven communication between services via src/events/
...
```

Where other models would need you to select specific files or summarize first, Gemini can ingest everything at once. This is especially valuable for:

- **Codebase-wide refactoring analysis** — understand all the implications before changing anything
- **Documentation generation** — read all the code, produce comprehensive docs
- **Migration planning** — analyze an entire legacy codebase to plan the move
- **Cross-file dependency mapping** — trace how data flows through the whole system

## Multimodal Capabilities

Gemini processes images, video, and audio natively. In Amplifier, this means you can mix media types freely in your conversations.

### Image Understanding

> Look at this architecture diagram and list all the services and their connections

```
The diagram shows six microservices:
1. API Gateway → Auth Service, User Service
2. Auth Service → Redis (session store)
3. User Service → PostgreSQL, Notification Service
4. Notification Service → Email Provider, SMS Provider
5. All services → Logging Service (via message queue)
```

### Video Analysis

> Watch this screen recording and identify where the UI bug occurs

```
At 0:42 in the recording, the dropdown menu renders behind the
modal overlay. The z-index stack is broken — the modal has
z-index: 1000 but the dropdown inherits z-index: auto from its
parent container instead of getting its own stacking context.
```

Video analysis is uniquely powerful for:
- Bug reproduction from screen recordings
- UX review of interaction flows
- Understanding deployment processes from recorded walkthroughs

### Image Generation

Gemini can generate images directly, which is useful for quick visual mockups and prototyping:

> Generate a wireframe of a login page with email, password, and social login buttons

```
[Gemini generates a wireframe image]

Here's a basic wireframe showing:
- Email input field at top
- Password field with show/hide toggle
- "Log In" primary button
- Divider with "or continue with"
- Google, GitHub, Apple social login buttons
- "Forgot password?" and "Sign up" links at bottom
```

This lets you iterate on visual ideas without leaving Amplifier or switching to a design tool.

## Multi-Provider Routing with Gemini

Gemini fits naturally into a routing matrix alongside other providers. A common pattern leverages Gemini's long context and multimodal strengths:

```yaml
providers:
  anthropic:
    api_key: ${ANTHROPIC_API_KEY}
  google:
    api_key: ${GOOGLE_API_KEY}

routing:
  coding: anthropic/claude-sonnet-4-20250514
  fast: google/gemini-2.5-flash
  research: google/gemini-2.5-pro
  vision: google/gemini-2.5-pro
  reasoning: anthropic/claude-sonnet-4-20250514
```

This setup uses Claude for core coding, Gemini Pro for research and vision tasks that benefit from the long context window, and Gemini Flash for quick operations.

## Tips

- **Use the context window.** Gemini's biggest advantage is processing massive inputs. Don't chunk what you can send whole — feed it entire files, full documentation, complete test suites.
- **Flash for speed, Pro for depth.** Flash is fast and cheap. Pro is thorough and powerful. Pick based on the task, not habit.
- **Multimodal is native.** You don't need to convert images to text descriptions or transcribe videos. Send the media directly and let Gemini process it in its original form.
- **top_k for focused output.** Gemini's `top_k` parameter gives you finer control over sampling. For code generation, `top_k: 20` with low temperature produces clean, predictable results.
- **Pair with Claude for coding.** Gemini excels at analysis and understanding; Claude excels at code generation. A routing matrix that plays to both strengths is more powerful than either alone.
- **Watch token costs on large contexts.** Sending 500K tokens of context is powerful but not free. Be deliberate about what you include — sometimes selecting the right files beats sending everything.

## Next Steps

- See the [Provider Index](./index.md) for the full routing configuration and multi-provider setup
- Compare with the [Anthropic Provider](./anthropic.md) and [OpenAI Provider](./openai.md) to understand where each model excels
- Learn about [Routing](../concepts/routing.md) to automate model selection across your workflow
