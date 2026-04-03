---
id: building-apps
type: app
title: "Building Your Own App"
---

# Building Your Own App

Amplifier isn't just a CLI tool you interact with at a terminal. It's a foundation you can build on. The same kernel that powers `amp` -- session management, module loading, event streaming, tool execution -- is available as a Python library. If you want to build a chatbot, an automated code reviewer, a voice assistant, or a CI integration that uses Amplifier's capabilities, you build it as an application on top of the Foundation library.

This page walks through the architecture, the key APIs, and the process of going from "I want to build something" to a working application.

## The Architecture

Every Amplifier application sits at the top of a four-layer stack:

```
Your Application
    |
    v
Foundation Library (amplifier-foundation)
    |
    v
Kernel (amplifier-core)
    |
    v
Modules (providers, tools, orchestrators, hooks, context)
```

**Your application** makes decisions about *what* to do -- which bundle to load, what prompts to send, how to present results. **Foundation** handles the heavy lifting -- loading bundles, resolving module dependencies, compiling everything into a mount plan, and giving you a clean `AmplifierSession` to work with. **The kernel** manages session lifecycle, coordinates modules, dispatches events, and runs the agent loop. **Modules** provide the actual capabilities -- the LLM provider, the tools, the orchestrator, the hooks.

The key insight: your application only talks to Foundation. Foundation talks to the kernel. The kernel talks to modules. Each layer has a clean boundary.

## Key Foundation Components

Three components from Foundation do most of the work in any application:

**`load_bundle()`** -- Takes a bundle path (local directory, git URL, or registry name) and returns a compiled bundle object. This is where module resolution, behavior merging, and include-chain processing happen.

**`AmplifierSession`** -- The main runtime object. You create a session from a loaded bundle, send prompts to it, and receive responses. Sessions manage their own module lifecycle -- mounting on entry, cleaning up on exit.

**Module loading** -- Foundation discovers and loads modules by name from the bundle's dependency chain. You declare `tool-bash` in your bundle YAML; Foundation finds the module, validates its contract, and registers it with the kernel.

## The Basic Pattern

The simplest possible Amplifier application loads a bundle, creates a session, and runs a prompt:

```python
import asyncio
from amplifier_foundation import load_bundle, AmplifierSession

async def main():
    # Load a bundle from a local path
    bundle = await load_bundle("./my-bundle")

    # Create a session and run a conversation
    async with AmplifierSession(bundle) as session:
        response = await session.execute("List all Python files in this project")
        print(response.text)

asyncio.run(main())
```

That's a complete application. `load_bundle()` compiles the bundle into a mount plan. `AmplifierSession` mounts all modules (provider, tools, orchestrator, hooks), sends the prompt through the orchestrator, and returns the result. The `async with` block handles cleanup automatically.

> What happens under the hood?

When `load_bundle()` runs, Foundation reads the bundle YAML, follows any `includes:` chains, merges behaviors, resolves `@mention` paths, and produces a flat mount plan. When `AmplifierSession` enters the `async with` block, it mounts every module in that plan. When you call `session.execute()`, the orchestrator drives the loop: prompt to provider, process tool calls, fire hooks, return the response.

## Loading Bundles

Bundles can come from several sources. Foundation handles all of them:

```python
# From a local directory
bundle = await load_bundle("./my-bundle")

# From a git URL
bundle = await load_bundle("git+https://github.com/microsoft/amplifier-foundation@main")

# From the registry (if configured)
bundle = await load_bundle("foundation")
```

You can also compose bundles programmatically by loading multiple and merging them:

```python
base = await load_bundle("foundation")
custom = await load_bundle("./my-overrides")
merged = base.compose(custom)

async with AmplifierSession(merged) as session:
    response = await session.execute("What tools do I have?")
```

This is how thin bundles work under the hood -- Foundation loads the include chain, composes each layer, and produces a single merged configuration.

## Advanced Integration Patterns

Once you're comfortable with the basic pattern, Foundation supports more sophisticated approaches.

### Custom Orchestrators

If the default agent loop doesn't fit your use case, you can provide a custom orchestrator -- say, a pipeline that always runs specific tools in sequence rather than letting the LLM choose:

```python
class PipelineOrchestrator:
    async def run_turn(self, coordinator, messages):
        result = await coordinator.call_tool("grep", {"pattern": "TODO", "path": "src/"})
        return await coordinator.complete(
            messages + [{"role": "user", "content": f"Summarize these TODOs: {result}"}]
        )
```

Register it in your bundle, and the kernel uses your orchestrator instead of the default loop.

### Event Streaming

For real-time applications -- chat UIs, dashboards, voice interfaces -- you need events as they happen, not a final response:

```python
async with AmplifierSession(bundle) as session:
    async for event in session.stream("Refactor the auth module"):
        if event.type == "token":
            print(event.data, end="", flush=True)
        elif event.type == "tool.called":
            print(f"\n[Tool: {event.tool}] {event.args}")
        elif event.type == "turn.completed":
            print("\n--- Turn complete ---")
```

Events are the same ones that hooks subscribe to -- `session.started`, `turn.started`, `tool.called`, `tool.result`, `token`, `turn.completed`. Streaming gives your application access to the full event firehose.

### Custom Tools

Your application can register tools that exist only in its context -- no module required:

```python
async with AmplifierSession(bundle) as session:
    @session.tool("check_inventory")
    async def check_inventory(product_id: str) -> str:
        """Check current inventory for a product."""
        count = await my_database.get_inventory(product_id)
        return f"Product {product_id}: {count} units in stock"

    response = await session.execute("How many widgets do we have in stock?")
```

The LLM sees `check_inventory` alongside all the bundle's tools and can call it like any other.

## Testing Your App

Testing an Amplifier application follows a three-stage progression: unit tests, local override, and shadow environments.

### Unit Tests

Test your application logic in isolation by mocking the session:

```python
async def test_inventory_check():
    mock_session = MockAmplifierSession()
    mock_session.set_response("Product W-100: 42 units in stock")

    app = InventoryApp(session=mock_session)
    result = await app.check_stock("W-100")

    assert "42 units" in result
    mock_session.assert_called_with("How many widgets do we have in stock?")
```

This tests your application's logic without touching the kernel, provider, or any real tools.

### Local Override

Test with the real kernel but a local bundle that overrides expensive components:

```python
# test-bundle/bundle.yaml -- uses a fast local model
bundle:
  name: test-override
  version: 1.0.0

includes:
  - bundle: ./my-bundle

providers:
  - module: provider-ollama
    config:
      model: llama3.1
```

```bash
pytest tests/ --bundle ./test-bundle
```

This exercises the full stack -- real module loading, real tool execution, real orchestration -- but with a cheaper provider for speed.

### Shadow Environments

For integration testing against production-like conditions, use a shadow workspace:

```bash
# Create an isolated copy of your project
cp -r my-app my-app-shadow
cd my-app-shadow

# Run your integration tests
python -m pytest tests/integration/ --live
```

Shadow environments let you test with real files, real git history, and real tools without risking your actual project. See [Shadow Workspace](../dev-setup/shadow-workspace.md) for the full setup.

## A Complete Example

Here's a minimal but complete application -- a CLI tool that loads a bundle and runs a multi-turn conversation:

```python
import asyncio
import sys
from amplifier_foundation import load_bundle, AmplifierSession

async def main():
    bundle_path = sys.argv[1] if len(sys.argv) > 1 else "foundation"
    bundle = await load_bundle(bundle_path)

    async with AmplifierSession(bundle) as session:
        print("Amplifier session ready. Type 'quit' to exit.\n")

        while True:
            user_input = input("> ")
            if user_input.strip().lower() == "quit":
                break

            async for event in session.stream(user_input):
                if event.type == "token":
                    print(event.data, end="", flush=True)
                elif event.type == "tool.called":
                    print(f"\n[Tool: {event.tool}]")

            print()  # Newline after response

asyncio.run(main())
```

Run it:

```bash
python my_app.py ./my-bundle
```

```
Amplifier session ready. Type 'quit' to exit.

> What files are in this project?
[Tool: glob]
Found 23 files: src/main.py, src/utils.py, tests/test_main.py, ...

> Run the tests
[Tool: bash]
===== 15 passed in 3.2s =====

All 15 tests pass. The test suite covers...
```

Thirty lines of Python, and you have a working conversational assistant with full tool access, streaming output, and multi-turn memory.

## Next Steps

- Understand the [Architecture](../concepts/architecture.md) that your application sits on top of
- Learn about [Bundles](../concepts/bundles.md) to configure what your app can do
- See [MCP Integration](./mcp.md) for connecting external tool servers to your app
- Read [Creating Custom Bundles](../advanced/custom-bundle.md) for packaging your app's configuration
