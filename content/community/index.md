---
id: community-bundles
type: community
title: "Community Bundles"
---

# Community Bundles

Amplifier ships with a rich set of official modules and bundles maintained by the core team. But the ecosystem doesn't stop there. A growing community of developers is building providers for new AI services, tools for specialized workflows, bundles for entire domains, and hooks that change how you interact with the agent. This page shows you how to find, use, and publish community contributions.

## What You'll Learn

By the end of this page, you'll know how to:

- Find community bundles, modules, and skills
- Evaluate community contributions for quality and trust
- Install a community bundle into your configuration
- Publish your own work for others to use

---

## Where to Find Community Contributions

The canonical directory of everything in the Amplifier ecosystem — official and community — lives in one place: the **MODULES.md** file in the main Amplifier repository.

> Where do I find community bundles?

```
[Tool: web_fetch] https://github.com/microsoft/amplifier/blob/main/docs/MODULES.md
```

This file is the ecosystem catalog. It lists every known component: core infrastructure, official modules, and community contributions organized by type. When someone builds something new for Amplifier, they submit a pull request to add it here.

You can also browse GitHub directly. Community projects follow a consistent naming convention:

| Type | Naming Pattern | Example |
|------|---------------|---------|
| **Bundle** | `amplifier-bundle-{name}` | `amplifier-bundle-memory` |
| **Module** | `amplifier-module-{type}-{name}` | `amplifier-module-provider-bedrock` |
| **App** | `amplifier-app-{name}` | `amplifier-app-voice` |

> Search GitHub for community providers

```
[Tool: web_search] "amplifier-module-provider" github
```

The naming convention makes discovery straightforward. If you're looking for a Perplexity provider, search for `amplifier-module-provider-perplexity`. Looking for a memory bundle? Try `amplifier-bundle-memory`.

---

## Popular Community Contributions

The community has built some impressive extensions. Here's a sampling across the major categories.

### Community Providers

These connect Amplifier to AI backends beyond the official set:

| Provider | What It Does | Author |
|----------|-------------|--------|
| **provider-bedrock** | AWS Bedrock with cross-region inference for Claude models | @brycecutt-msft |
| **provider-perplexity** | Perplexity AI chat completions with sonar models | @colombod |
| **provider-openai-realtime** | Native speech-to-speech via OpenAI Realtime API | @robotdad |

Because all providers follow the same `complete(request) → response` contract, swapping in a community provider works exactly like swapping between official ones. Your tools, hooks, and orchestrator don't change — only the AI backend does.

### Community Bundles

Bundles package entire experiences — agents, context, tools, and configuration:

| Bundle | What It Does | Author |
|--------|-------------|--------|
| **memory** | Persistent memory with automatic capture and progressive disclosure | @michaeljabbour |
| **perplexity** | Deep web research with citations and cost-aware guidance | @colombod |
| **deepwiki** | Ask questions about any public GitHub repo via DeepWiki MCP | @colombod |
| **browser** | Browser automation with JS rendering, auth flows, and form filling | @samueljklee |
| **expert-cookbook** | State-of-the-art workflows and techniques for Amplifier | @DavidKoleczek |
| **tui-tester** | AI-assisted testing for TUI and terminal applications | @colombod |
| **web-ux-dev** | Visual regression testing, console debugging, pre-commit checks | @colombod |

### Community Tools and Hooks

Individual modules that add specific capabilities:

| Module | What It Does | Author |
|--------|-------------|--------|
| **tool-memory** | Persistent fact storage across sessions | @michaeljabbour |
| **tool-rlm** | Process 10M+ token contexts via sandboxed Python REPL | @michaeljabbour |
| **tool-youtube-dl** | Download audio and video from YouTube | @robotdad |
| **hooks-explanatory** | Educational ★ Insight blocks in responses | @michaeljabbour |
| **hooks-concise-display** | Cleaner, more condensed terminal output | @obra |
| **hooks-event-broadcast** | Transport-agnostic event broadcasting for streaming UIs | @michaeljabbour |

---

## Using a Community Bundle

Installing a community bundle follows the same pattern as any bundle — you point Amplifier at a git URL:

```bash
# Add a community bundle to your registry
amplifier bundle add git+https://github.com/michaeljabbour/amplifier-bundle-memory@main

# Activate it
amplifier bundle use memory
```

You can also reference community bundles directly in your bundle YAML when composing configurations:

```yaml
---
bundle:
  name: my-research-setup
  version: 1.0.0

extends:
  - foundation
  - source: git+https://github.com/colombod/amplifier-bundle-perplexity@main
---

# Research assistant with deep web search
Use Perplexity for research tasks requiring citations.
```

For individual modules — a community provider, tool, or hook — add them in your bundle's module section:

```yaml
providers:
  - module: provider-bedrock
    source: git+https://github.com/brycecutt-msft/amplifier-module-provider-bedrock@main
```

> Add the memory bundle to my current setup

```
[Tool: bash] amplifier bundle add git+https://github.com/michaeljabbour/amplifier-bundle-memory@main
Added bundle: memory (v1.2.0)

[Tool: bash] amplifier bundle use memory
Active bundle: memory
```

That's it. The kernel discovers the bundle, loads its modules, and the new capabilities appear in your session.

---

## Publishing Your Own

Built something useful? Here's how to share it with the ecosystem.

### 1. Create a Repository

Follow the naming convention so others can discover your work:

```
amplifier-bundle-{your-name}     # for bundles
amplifier-module-{type}-{name}   # for modules (e.g., tool-*, provider-*, hooks-*)
```

### 2. Structure It Properly

For a bundle, include a `bundle.yaml` with clear metadata:

```yaml
bundle:
  name: my-awesome-bundle
  description: What this bundle does in one sentence
  version: 1.0.0
  author: your-github-handle
```

For a module, include a `pyproject.toml` with the entry point:

```toml
[project.entry-points."amplifier.modules"]
tool-my-thing = "my_tool:mount"
```

### 3. Test Thoroughly

Make sure your contribution works with the current Amplifier release. Include tests, and document any prerequisites or environment variables.

### 4. Add to the Catalog

Submit a pull request to the main [amplifier](https://github.com/microsoft/amplifier) repository, adding your entry to `docs/MODULES.md` under the appropriate community section. Include your GitHub handle, a concise description, and a link to the repository.

---

## Quality and Trust

Community contributions are powerful, but they come with responsibility — both for authors and users.

> **Security consideration**: Community modules execute code in your environment with full access to your filesystem, network, and credentials. Only use modules from sources you trust.

### For Users

Before installing a community module:

1. **Read the source code.** Community modules aren't reviewed by the Amplifier core team. Skim the `mount()` function and understand what it registers.
2. **Check the author's reputation.** Look at their GitHub profile, other contributions, and activity in the Amplifier community.
3. **Pin to a specific commit.** Use `@sha` instead of `@main` for production setups to avoid surprise changes.
4. **Test in isolation.** Try the module in a scratch session before adding it to your daily workflow.

```bash
# Pin to a specific commit for safety
amplifier bundle add git+https://github.com/author/amplifier-bundle-example@a1b2c3d
```

### For Authors

When publishing community modules:

1. **Document clearly.** Explain what your module does, what permissions it needs, and any external dependencies.
2. **Handle errors gracefully.** A module should never crash the kernel. Return error information rather than raising unhandled exceptions.
3. **Follow the contracts.** Use Amplifier's protocols and structural typing. If your module passes protocol compliance, it's a good citizen.
4. **Version your releases.** Use semantic versioning and tag releases so users can pin to stable versions.

---

## Key Takeaways

1. **MODULES.md is the catalog.** The canonical directory of all Amplifier components — official and community — lives in the main repository's `docs/MODULES.md`.

2. **Naming conventions enable discovery.** Follow `amplifier-bundle-*` and `amplifier-module-*` patterns so others can find your work.

3. **Installation is a git URL.** Community bundles install the same way as official ones — `amplifier bundle add` with a git URL.

4. **Trust requires verification.** Community code runs with full access to your environment. Read the source, check the author, pin versions.

5. **Publishing is a PR.** Build it, name it correctly, test it, and submit a pull request to add it to the ecosystem catalog.

6. **The ecosystem is growing.** Community providers, bundles, tools, hooks, and apps are expanding what Amplifier can do in every direction.
