# Amplifier Tutorial Website - Research & Plan

## Executive Summary

This document outlines the plan for creating a **narrative tutorial website** for Amplifier that:
- Explains what Amplifier is and how it differs from competitors
- Provides a smooth "out of box" experience for new users
- Offers progressive learning paths through all capabilities
- Can be updated incrementally without full regeneration
- Uses recipes/tools for both generation and maintenance

---

## Part 1: Research Findings

### 1.1 The Amplifier Ecosystem (Complete Map)

#### Core Infrastructure
| Component | Repository | Purpose |
|-----------|-----------|---------|
| **Entry Point** | microsoft/amplifier | User-facing docs, installation, ecosystem overview |
| **Kernel** | microsoft/amplifier-core | Ultra-thin core (~2,600 lines), module loading, events |
| **Foundation** | microsoft/amplifier-foundation | Bundle composition, reference bundles, 21 examples |

#### Official Bundles
| Bundle | Repository | Purpose |
|--------|-----------|---------|
| **recipes** | amplifier-bundle-recipes | Multi-step workflow orchestration |
| **design-intelligence** | amplifier-bundle-design-intelligence | 7 specialized design agents |
| **lsp** | amplifier-bundle-lsp | Language Server Protocol base |
| **lsp-python** | amplifier-bundle-lsp-python | Python code intelligence (Pyright) |
| **lsp-typescript** | amplifier-bundle-lsp-typescript | TypeScript/JS code intelligence |

#### Official Modules
| Type | Modules |
|------|---------|
| **Providers** | anthropic, openai, azure-openai, gemini, ollama, vllm, mock |
| **Tools** | filesystem, bash, web, search, task, todo, skills |
| **Orchestrators** | loop-basic, loop-streaming, loop-events |
| **Context** | context-simple, context-persistent |
| **Hooks** | logging, redaction, approval, streaming-ui, status-context, todo-reminder, etc. |

#### Community Resources (robotdad)
| Component | Repository | Tutorial Value |
|-----------|-----------|----------------|
| **Skills Package** | robotdad/skills | Playwright, curl, image-vision - debugging skills |
| **MCP Integration** | amplifier-module-tool-mcp | Production-ready MCP server integration |
| **Blog Creator** | amplifier-bundle-blog-creator | Multi-agent workflow example |
| **Spec-Kit** | amplifier-collection-spec-kit | 8-agent SDD methodology |
| **DDD Collection** | amplifier-collection-ddd | Document-driven development |
| **Voice App** | amplifier-app-voice | Voice-first Amplifier interface |

#### Developer Tools (bkrabach)
| Tool | Repository | Tutorial Value |
|------|-----------|----------------|
| **CLI Tools** | amplifier-cli-tools | `amplifier-dev setup`, workspace management |
| **Shadow Workspace** | amplifier-shadow | Isolated Docker environments for safe AI experimentation |
| **Project Template** | ai-code-project-template | AI-first project structure with `/prime` command |
| **Trace Viewer** | claude-trace-viewer | Session debugging visualization |
| **Remote Dev Guide** | (in cli-tools) | Tailscale + Mosh + tmux mobile development |

### 1.2 Key Differentiators (vs Claude Code / GitHub Copilot)

| Aspect | Claude Code / Copilot | Amplifier |
|--------|----------------------|-----------|
| **Architecture** | Monolithic, closed | Modular kernel + swappable modules |
| **Provider Lock-in** | Single vendor | Multi-provider (Anthropic, OpenAI, Azure, Ollama, etc.) |
| **Extensibility** | Limited plugins | Full module system (tools, hooks, orchestrators) |
| **Workflows** | Ad-hoc prompts | Declarative recipes with state, approvals, resumption |
| **Customization** | Settings only | Bundles compose any capability mix |
| **Multi-Agent** | Single agent | Native sub-agent spawning with task tool |
| **Transparency** | Black box | Event-first observability, JSONL logs |
| **Local Control** | Cloud-dependent | Ollama support, local-first possible |
| **Composability** | N/A | Behaviors, skills, bundles, agents all compose |

### 1.3 Core Concepts for Tutorial

1. **Bundles** - Composable configuration packages (the "what")
2. **Modules** - Swappable capabilities (providers, tools, hooks, orchestrators)
3. **Agents** - Specialized AI configurations as sub-sessions
4. **Recipes** - Declarative multi-step workflows with state
5. **Skills** - Reusable knowledge packages (Anthropic format)
6. **Behaviors** - Capability add-ons that bundle agents + context
7. **Hooks** - Lifecycle observers for control and observability
8. **Events** - Structured observability (session:*, turn:*, tool:*, provider:*)

---

## Part 2: Tutorial Structure

### 2.1 Learning Paths (Progressive)

```
Level 0: "What Is This?" (5 min)
├── What is Amplifier?
├── How is it different from Claude Code / Copilot?
├── The Linux Kernel Philosophy
└── When would I use this?

Level 1: "Quick Start" (15 min)
├── Install (uv tool install)
├── Configure provider (amplifier init)
├── First conversation
├── Key commands (/help, /tools, /agents)
└── Install a bundle (recipes)

Level 2: "Core Concepts" (30 min)
├── Understanding Bundles
├── Modules: The building blocks
├── Agents: Specialized helpers
├── Skills: Reusable knowledge
└── Recipes: Orchestrated workflows

Level 3: "Developer Setup" (20 min)
├── Using amplifier-dev CLI tools (bkrabach)
├── Workspace organization
├── Remote development (Tailscale + Mosh + tmux)
├── Shadow workspaces for safe experimentation
└── Debugging with trace viewer

Level 4: "Exploring Capabilities" (modular sections)
├── All Official Tools (deep dives)
├── All Official Bundles (deep dives)
├── All Available Skills (with examples)
├── Community Resources (robotdad, etc.)
├── Example Recipes (11 provided)
└── Foundation Examples (21 numbered)

Level 5: "Building Your Own" (advanced)
├── Creating a custom bundle
├── Writing a new tool module
├── Authoring an agent
├── Creating a skill
├── Building a recipe
└── Multi-agent architectures
```

### 2.2 Module-Based Content Structure

To enable incremental updates, each content piece is a **module**:

```
content/
├── meta/
│   ├── site-config.yaml       # Site-wide configuration
│   └── navigation.yaml        # Menu structure
├── concepts/
│   ├── bundles.md
│   ├── modules.md
│   ├── agents.md
│   ├── recipes.md
│   ├── skills.md
│   └── hooks.md
├── quickstart/
│   ├── install.md
│   ├── first-run.md
│   ├── first-bundle.md
│   └── key-commands.md
├── tools/
│   ├── _index.yaml            # Tool registry (auto-generated)
│   ├── tool-filesystem.md
│   ├── tool-bash.md
│   ├── tool-web.md
│   └── ...
├── bundles/
│   ├── _index.yaml            # Bundle registry (auto-generated)
│   ├── foundation.md
│   ├── recipes.md
│   ├── lsp-python.md
│   └── ...
├── skills/
│   ├── _index.yaml
│   ├── playwright.md
│   ├── curl.md
│   └── image-vision.md
├── recipes/
│   ├── _index.yaml
│   ├── code-review.md
│   ├── dependency-upgrade.md
│   └── ...
├── examples/
│   ├── _index.yaml
│   ├── 01-hello-world.md
│   └── ...
├── dev-setup/
│   ├── cli-tools.md
│   ├── shadow-workspace.md
│   ├── remote-dev.md
│   └── trace-viewer.md
└── advanced/
    ├── custom-bundle.md
    ├── custom-tool.md
    ├── custom-agent.md
    └── multi-agent.md
```

### 2.3 Content Module Schema

Each content module follows a standard frontmatter schema:

```yaml
---
# content/tools/tool-filesystem.md
id: tool-filesystem
type: tool                    # concept | quickstart | tool | bundle | skill | recipe | example | advanced
title: "Filesystem Tool"
source_repo: microsoft/amplifier-module-tool-filesystem
source_path: README.md        # Where to fetch current docs
last_synced: 2026-01-07
dependencies:
  - concepts/modules.md       # Prerequisite reading
related:
  - tools/tool-bash.md
  - bundles/foundation.md
exercises:
  - id: read-a-file
    title: "Read a file"
    difficulty: beginner
    instructions: "Use the filesystem tool to read package.json"
    validation: "Check that file contents are displayed"
---

# Filesystem Tool

[Narrative content here, enriched with tutorial context]
```

---

## Part 3: Generation & Maintenance Architecture

### 3.1 Recipe-Based Approach

**Key Insight**: Use Amplifier recipes to build and maintain the Amplifier tutorial!

#### Research Recipe (`recipes/research-component.yaml`)
```yaml
name: research-component
description: Research a single component and generate/update its content module

steps:
  - id: fetch-source
    agent: foundation:web-research
    instruction: |
      Fetch the current documentation for {{component_repo}}/{{component_path}}.
      Extract: purpose, API, usage examples, common patterns.

  - id: check-existing
    agent: foundation:file-ops
    instruction: |
      Check if content/{{component_type}}/{{component_id}}.md exists.
      If it exists, load it and note the last_synced date.

  - id: generate-content
    agent: foundation:modular-builder
    instruction: |
      Generate or update the content module for {{component_id}}.
      - Follow the content module schema
      - Preserve any custom tutorial content (marked with <!-- CUSTOM -->)
      - Update the source-derived sections
      - Create progressive exercises

  - id: update-index
    agent: foundation:file-ops
    instruction: |
      Update content/{{component_type}}/_index.yaml with this component.
```

#### Full Ecosystem Scan Recipe (`recipes/scan-ecosystem.yaml`)
```yaml
name: scan-ecosystem
description: Discover all components across Amplifier ecosystem

steps:
  - id: fetch-modules-md
    agent: foundation:web-research
    instruction: Fetch microsoft/amplifier/docs/MODULES.md

  - id: parse-components
    agent: foundation:zen-architect
    instruction: |
      Parse MODULES.md into structured component list:
      - repos, modules, bundles, tools, providers, hooks
      
  - id: discover-community
    agent: foundation:web-research
    instruction: |
      Scan GitHub for additional Amplifier components:
      - robotdad repos with "amplifier" prefix
      - bkrabach repos with "amplifier" prefix
      - Search "amplifier-bundle-*" and "amplifier-module-*"

  - id: generate-manifest
    agent: foundation:file-ops
    instruction: |
      Create/update .planning/ecosystem-manifest.yaml with all discovered components.
      Include: name, repo, type, description, last_checked
```

#### Refresh Tutorial Recipe (`recipes/refresh-tutorial.yaml`)
```yaml
name: refresh-tutorial
description: Update all tutorial content from source repos

steps:
  - id: load-manifest
    agent: foundation:file-ops
    instruction: Load .planning/ecosystem-manifest.yaml

  - id: refresh-each
    foreach: "{{manifest.components}}"
    parallel: 3
    steps:
      - id: research
        recipe: ./research-component.yaml
        context:
          component_id: "{{item.id}}"
          component_repo: "{{item.repo}}"
          component_type: "{{item.type}}"
          component_path: "{{item.source_path}}"

  - id: regenerate-indices
    agent: foundation:file-ops
    instruction: Regenerate all _index.yaml files from content

  - id: update-navigation
    agent: foundation:file-ops
    instruction: Update meta/navigation.yaml from content structure
```

### 3.2 Tool Support

Create a helper tool for common operations:

```python
# tools/tutorial-tool.py
"""
Tool for tutorial maintenance operations.

Commands:
- scan: Discover ecosystem components
- status: Show content freshness (last_synced vs repo activity)
- refresh <component>: Update a single component
- refresh-all: Update all components
- validate: Check for broken links, missing prereqs
- build: Generate static site from content modules
"""
```

### 3.3 Update Workflow

```
User says: "Go look through the repo and update the tutorial"
    │
    ▼
1. Run scan-ecosystem recipe → Update ecosystem-manifest.yaml
    │
    ▼
2. Compare last_synced dates with repo activity
    │
    ▼
3. Run refresh-tutorial recipe (only for stale components)
    │
    ▼
4. Validate content (links, prereqs, exercises)
    │
    ▼
5. Build static site (if requested)
```

---

## Part 4: Site Generation

### 4.1 Technology Choice

**Recommendation: Static site generator** that can:
- Read markdown with frontmatter
- Generate navigation from structure
- Support search
- Handle code syntax highlighting
- Deploy to GitHub Pages

Options:
1. **Docusaurus** (React) - Great for docs, versioning support
2. **VitePress** (Vue) - Fast, simple, good for tutorials
3. **Astro** (Islands) - Flexible, great DX
4. **MkDocs Material** (Python) - Simple, beautiful

**Suggested: VitePress or Docusaurus** - both handle the modular content structure well.

### 4.2 Build Pipeline

```yaml
# .github/workflows/build-tutorial.yaml
on:
  push:
    paths: ['content/**']
  workflow_dispatch:
    inputs:
      refresh:
        description: 'Refresh from source repos first?'
        type: boolean

jobs:
  build:
    steps:
      - if: inputs.refresh
        run: amplifier recipes execute recipes/refresh-tutorial.yaml
      
      - run: npm run build  # VitePress/Docusaurus
      
      - uses: peaceiris/actions-gh-pages@v3
```

---

## Part 5: Content Inventory (What to Create)

### 5.1 Priority 1: Core Path (New User Experience)

| Content | Source | Status |
|---------|--------|--------|
| What is Amplifier? | New (synthesize from philosophy docs) | TODO |
| How it differs (Claude Code, Copilot) | New (competitive positioning) | TODO |
| Install & Setup | USER_ONBOARDING.md | TODO |
| First Conversation | New (walkthrough) | TODO |
| Key Commands | USER_ONBOARDING.md | TODO |
| Install Your First Bundle | New (recipes bundle example) | TODO |

### 5.2 Priority 2: Core Concepts

| Content | Source | Status |
|---------|--------|--------|
| Bundles | BUNDLE_GUIDE.md | TODO |
| Modules | MODULES.md + contracts | TODO |
| Agents | AGENT_AUTHORING.md | TODO |
| Recipes | RECIPE_SCHEMA.md | TODO |
| Skills | tool-skills README | TODO |
| Hooks | HOOKS_API.md | TODO |

### 5.3 Priority 3: Deep Dives (Auto-Generated)

| Category | Count | Source |
|----------|-------|--------|
| Tools | 7+ | Individual module READMEs |
| Bundles | 5+ | Individual bundle READMEs |
| Providers | 7 | Individual provider READMEs |
| Skills | 3+ | robotdad/skills |
| Example Recipes | 11 | amplifier-bundle-recipes/examples |
| Foundation Examples | 21 | amplifier-foundation/examples |

### 5.4 Priority 4: Developer Resources

| Content | Source | Status |
|---------|--------|--------|
| amplifier-dev CLI | bkrabach/amplifier-cli-tools | TODO |
| Shadow Workspaces | bkrabach/amplifier-shadow | TODO |
| Remote Dev Setup | REMOTE_MOBILE_DEV.md | TODO |
| Trace Viewer | bkrabach/claude-trace-viewer | TODO |
| AI Project Template | bkrabach/ai-code-project-template | TODO |

---

## Part 6: Proposed Next Steps

### Phase 1: Foundation (This Week)
1. Create the content directory structure
2. Create the ecosystem-manifest.yaml from research
3. Write core content (What is Amplifier, Differentiators, Install)
4. Set up site generator (VitePress recommended)

### Phase 2: Recipes & Tools (Next Week)
1. Implement scan-ecosystem recipe
2. Implement research-component recipe
3. Implement refresh-tutorial recipe
4. Create tutorial-tool for CLI operations

### Phase 3: Content Generation (Week 3)
1. Run scan-ecosystem to discover all components
2. Generate content modules for all tools, bundles, skills
3. Write exercises for each section
4. Create navigation and search

### Phase 4: Polish & Deploy (Week 4)
1. Review and enhance auto-generated content
2. Add interactive examples (CodeSandbox, etc.)
3. Set up GitHub Actions for auto-refresh
4. Deploy to GitHub Pages

---

## Appendix: Key Source Files

| Purpose | Location |
|---------|----------|
| Ecosystem catalog | microsoft/amplifier/docs/MODULES.md |
| User onboarding | microsoft/amplifier/docs/USER_ONBOARDING.md |
| Bundle guide | microsoft/amplifier-foundation/docs/BUNDLE_GUIDE.md |
| Agent authoring | microsoft/amplifier-foundation/docs/AGENT_AUTHORING.md |
| Recipe schema | microsoft/amplifier-bundle-recipes/docs/RECIPE_SCHEMA.md |
| Hook API | microsoft/amplifier-core/docs/HOOKS_API.md |
| Provider contract | microsoft/amplifier-core/docs/contracts/PROVIDER_CONTRACT.md |
| Tool contract | microsoft/amplifier-core/docs/contracts/TOOL_CONTRACT.md |
| Foundation examples | microsoft/amplifier-foundation/examples/ |
| Recipe examples | microsoft/amplifier-bundle-recipes/examples/ |
| Playwright skill | robotdad/skills/playwright/ |
| CLI tools | bkrabach/amplifier-cli-tools |
| Shadow workspace | bkrabach/amplifier-shadow |

---

*This plan is modular and designed for iteration. Let's discuss!*
