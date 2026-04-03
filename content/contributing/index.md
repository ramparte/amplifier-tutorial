---
id: contributing-overview
type: contributing
title: "Contributing Overview"
---

# Contributing to Amplifier

Amplifier is an open-source project with a modular architecture — and that modularity extends to how people contribute. You don't need to understand the entire system to make a meaningful impact. Build a tool module without touching the kernel. Write a skill without understanding orchestration. Fix a bug in a provider without knowing how hooks work. The architecture's clear boundaries mean you can contribute at whatever level matches your interest and expertise.

This page explains how the ecosystem is organized, where different types of contributions belong, and how to get your work reviewed and merged.

## Section Contents

| Page | Description |
|------|-------------|
| [Writing Modules](./writing-modules.md) | Build new tools, providers, hooks, and context modules |
| [Writing Bundles](./writing-bundles.md) | Package modules and context into shareable bundles |
| [Writing Skills](./writing-skills.md) | Create domain knowledge files for agent consumption |
| [Writing Agents](./writing-agents.md) | Define specialized agents with focused capabilities |

## What You'll Learn

By the end of this page, you'll understand:

- The repository hierarchy and where to contribute
- What types of contributions the project welcomes
- The pull request process and review guidelines
- Community standards and code of conduct

---

## The Repository Hierarchy

Amplifier isn't a single monolithic repository. It's an ecosystem of focused repositories, each with a clear responsibility. Understanding this hierarchy is the first step to knowing where your contribution belongs.

```
┌─────────────────────────────────────────────┐
│              amplifier (entry point)         │
│         Main repo, docs, MODULES.md         │
├─────────────────────────────────────────────┤
│           amplifier-core (kernel)            │
│        ~2,600 lines of Rust + PyO3          │
├─────────────────────────────────────────────┤
│        amplifier-foundation (library)        │
│     Bundle primitives, shared utilities      │
├──────────────┬──────────────────────────────┤
│  Applications │        Bundles              │
│  amplifier-   │  amplifier-bundle-*         │
│  app-cli      │  foundation, recipes,       │
│  app-log-     │  python-dev, skills ...     │
│  viewer       │                             │
├──────────────┴──────────────────────────────┤
│              Runtime Modules                 │
│  amplifier-module-provider-*                │
│  amplifier-module-tool-*                    │
│  amplifier-module-hooks-*                   │
│  amplifier-module-loop-*                    │
│  amplifier-module-context-*                 │
└─────────────────────────────────────────────┘
```

### Where Does My Contribution Go?

| I want to... | Contribute to |
|--------------|---------------|
| Add a new AI provider | New repo: `amplifier-module-provider-{name}` |
| Build a new tool | New repo: `amplifier-module-tool-{name}` |
| Create a bundle | New repo: `amplifier-bundle-{name}` |
| Write a skill | The relevant bundle's `skills/` directory |
| Fix a CLI bug | `amplifier-app-cli` |
| Fix a kernel bug | `amplifier-core` |
| Improve documentation | The repo that owns the content |
| Add to the ecosystem catalog | `amplifier` (main repo, `docs/MODULES.md`) |

---

## Repository Governance

Amplifier follows a set of repository awareness rules that prevent context poisoning and keep documentation honest. The core principles from `REPOSITORY_RULES.md`:

1. **Single source of truth.** Content lives in ONE place. Never duplicate documentation across repositories.
2. **Link, don't duplicate.** Other repos reference content via GitHub URLs, not by copying it.
3. **Respect awareness boundaries.** A repository can only reference another if it has a declared dependency on it.
4. **Dependency-based awareness.** Only reference what you actually depend on — listed in `pyproject.toml` or imported in code.

### The Awareness Rules in Practice

| Repository Type | Can Reference | Cannot Reference |
|----------------|---------------|------------------|
| **Kernel** (amplifier-core) | Entry point only | Libraries, modules, apps |
| **Libraries** | Core + entry + declared deps | Anything not in pyproject.toml |
| **Modules** | Core + entry point | Peer modules (never) |
| **Applications** | Anything they consume | — |
| **Entry point** (amplifier) | Everything | — |

The most common mistake for new contributors: a module referencing a peer module in its documentation. `tool-bash` cannot reference `tool-filesystem`, even casually. They don't know each other exists. The kernel wires them together — modules don't reach across.

> Why are these rules so strict?

Because AI tools (including Amplifier itself) load repository documentation as context. If a public module references a private repo, or if two modules cross-reference each other, the AI gets confused about what exists, what depends on what, and where things live. Clean boundaries make the whole ecosystem work.

---

## Types of Contributions

### Modules

The most common contribution. Build a new provider, tool, hook, orchestrator, or context module as its own repository. See [Writing a Module](./writing-modules.md) for the full walkthrough.

### Bundles

Package a set of modules, agents, context, and skills into a reusable configuration. Bundles are higher-level than modules — they define experiences rather than individual capabilities.

### Skills

Skills are markdown files that teach the agent domain knowledge. They live inside bundles, in the `skills/` directory. If you have expertise in a specific domain, writing a skill is one of the easiest and highest-impact contributions you can make.

### Documentation

Every repository has documentation that can be improved. Fix typos, clarify examples, add missing sections. Documentation PRs are just as valued as code PRs.

### Bug Fixes

Found a bug? File an issue first to discuss it, then submit a PR with the fix. Include a test that demonstrates the bug was fixed.

### Ecosystem Catalog

Built something new? Add it to `docs/MODULES.md` in the main amplifier repository. This is the single directory of all known Amplifier components.

---

## The PR Process

### Before You Start

1. **Check existing issues.** Someone may already be working on the same thing.
2. **Open an issue for discussion.** For non-trivial changes, discuss the approach before writing code. This saves everyone time.
3. **Read the repo's CONTRIBUTING.md.** Each repository may have specific guidelines beyond what's covered here.

### Writing Your PR

> I've fixed a bug in the bash tool. How do I submit it?

```
[Tool: bash] cd amplifier-module-tool-bash
[Tool: bash] git checkout -b fix/timeout-handling
[Tool: bash] git add -A && git commit -m "fix: handle timeout edge case when command exits during signal"
[Tool: bash] gh pr create --title "fix: handle timeout edge case" --body "Fixes #42. When a command exits during SIGTERM delivery, the tool now catches the race condition and returns the partial output instead of raising an unhandled exception."
```

### PR Guidelines

**Keep PRs focused.** One logical change per PR. If you're fixing a bug and also reformatting code, that's two PRs.

**Write descriptive commit messages.** Use conventional commits: `fix:`, `feat:`, `docs:`, `refactor:`, `test:`. The message should explain *why*, not just *what*.

**Include tests.** If you're fixing a bug, add a test that fails without the fix and passes with it. If you're adding a feature, add tests that cover the happy path and edge cases.

**Update documentation.** If your change affects behavior, update the relevant docs in the same PR.

**Respect the module boundary.** Your PR should not introduce cross-module dependencies. A tool module that imports from a hook module will be rejected.

### Code Review

Reviewers will check for:

- **Contract compliance** — Does your module follow its protocol?
- **Error handling** — Does it fail gracefully without crashing the kernel?
- **Isolation** — Does it respect module boundaries?
- **Tests** — Are the important paths covered?
- **Documentation** — Is the change documented?

Expect at least one round of review feedback. This is normal and healthy — it's how the codebase stays consistent.

---

## Community Standards

### Code of Conduct

Amplifier follows the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/). The short version: be respectful, be constructive, and assume good intent.

### Communication

- **GitHub Issues** — For bugs, feature requests, and design discussions
- **Pull Requests** — For code review and contribution
- **Discussions** — For open-ended questions and community conversations

### What Makes a Good Contribution

The best contributions share a few qualities:

1. **They solve a real problem.** The contributor encountered a genuine need and built a solution.
2. **They follow the architecture.** They use the module system, respect the contracts, and don't introduce hidden coupling.
3. **They're well-documented.** Other people can understand what the contribution does and how to use it.
4. **They include tests.** Automated verification that the contribution works as intended.
5. **They're focused.** Do one thing well rather than many things poorly.

---

## Getting Started

The fastest path to your first contribution depends on your interest:

| Interest | First Step |
|----------|-----------|
| **Building tools** | Read [Writing a Module](./writing-modules.md), then build a simple tool |
| **Domain expertise** | Write a skill for a domain you know well |
| **Bug hunting** | Browse open issues across the ecosystem repos |
| **Documentation** | Pick any page that confused you and make it clearer |
| **New provider** | Check if your preferred AI service has a provider; if not, build one |

Whatever you choose, the community is welcoming to newcomers. Ask questions in issues, start small, and build from there.

---

## Key Takeaways

1. **Amplifier is a multi-repo ecosystem.** Know where your contribution goes before you start — the hierarchy determines which repository to target.

2. **Awareness rules prevent chaos.** Modules can't reference peers, documentation lives in one place, and dependencies are explicit. These rules keep the ecosystem healthy.

3. **Modules are the most common contribution.** Each module is its own repo, its own package, and its own responsibility.

4. **PRs should be focused and tested.** One change, good commit messages, tests included, docs updated.

5. **Community standards are simple.** Be respectful, solve real problems, follow the architecture, document your work.
