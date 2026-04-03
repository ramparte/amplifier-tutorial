# Tutorial Site Update Plan -- April 2026

**Created:** 2026-04-02
**Status:** IN PROGRESS
**Last content update:** 2026-01-09 (all fallback-generated, never sourced from repos)
**Live site:** https://ramparte.github.io/amplifier-tutorial/

---

## Situation

The tutorial site has 46 pages generated on January 9, 2026 using "fallback mode"
(AI-generated without pulling from actual source repos). The Amplifier ecosystem
has grown significantly since then with 30+ new capabilities, bundles, tools, and
applications. The site needs a comprehensive update.

## Scope

**In scope:** All public Amplifier ecosystem repos (microsoft/amplifier-*) and
public community repos. Content that external users and new team members need.

**Out of scope (private/unreleased):**
- Dev Machines bundle (amplifier-bundle-dev-machine) -- private
- word4, botslack, book2, universal-app -- private standalone apps
- Expert Machine bundle -- private
- SOAR bundle -- private/research
- Trident bundle -- private/research
- Safeguard -- private/enterprise
- M365 Platform -- private/enterprise
- Session Sync daemon -- private infrastructure
- Amplifier Distro / Kepler -- private distribution
- OpenM365 worker -- private

## Strategy

Use the existing recipe infrastructure where possible. For new content, use
subagent delegation with the `generate-*` recipes or direct agent orchestration.
Track all work in this document.

---

## Phase 0: Infrastructure Fixes

Must complete before any content generation.

| # | Task | Status | Notes |
|---|------|--------|-------|
| 0.1 | Fix hardcoded WSL path in `update-tutorial.yaml` | DONE | Changed 5 locations from `/mnt/c/ANext/...` to relative paths |
| 0.2 | Fix hardcoded paths in bash steps of `update-tutorial.yaml` | DONE | `build-site`, `count-files`, `final-report` steps fixed |
| 0.3 | Fix `requirements.txt` vs `mkdocs.yml` mismatch | DONE | Removed unused `mkdocs-material` dependency |
| 0.4 | Clean up 7 orphaned screenshots in root | NOT STARTED | 7 PNGs totaling ~281KB, not referenced by content |
| 0.5 | Fix filename mismatch (nav vs content) | N/A | Verified: no mismatch exists, nav matches actual files |
| 0.6 | Verify recipe schema compatibility | NOT STARTED | `refresh-tutorial.yaml` uses `instruction:` vs `prompt:` |

---

## Phase 1: Ecosystem Scan & Manifest Update

Refresh the ecosystem manifest to reflect current state.

| # | Task | Status | Notes |
|---|------|--------|-------|
| 1.1 | Run `scan-ecosystem.yaml` recipe | DONE | 138 components found (2.5x growth from 55). Manifest updated. |
| 1.2 | Review scan corrections | DONE | LSP bundles deprecated -> python-dev/ts-dev/rust-dev. Old skills removed. Foundation bundle restructured. |
| 1.3 | Tag capabilities as public/private in manifest | NOT STARTED | Skip private repos in generation |

---

## Phase 2: Refresh Existing Content (24 pages)

Regenerate all existing pages with proper research (not fallback mode).
Use `research-component.yaml` or type-specific recipes.

### Quickstart Section (6 pages)

| # | Page | File | Status | Notes |
|---|------|------|--------|-------|
| 2.1 | What is Amplifier? | `quickstart/what-is-amplifier.md` | NOT STARTED | Update with current architecture (Rust kernel) |
| 2.2 | How It's Different | `quickstart/how-its-different.md` | NOT STARTED | Add new differentiators (modes, recipes, discovery) |
| 2.3 | Installation | `quickstart/installation.md` | NOT STARTED | Verify against current `amplifier` CLI |
| 2.4 | First Conversation | `quickstart/first-conversation.md` | NOT STARTED | Update example prompts and workflows |
| 2.5 | Key Commands | `quickstart/key-commands.md` | NOT STARTED | Add new commands, slash commands, modes |
| 2.6 | First Bundle | `quickstart/first-bundle.md` | NOT STARTED | Update bundle YAML format if changed |

### Core Concepts Section (7 pages)

| # | Page | File | Status | Notes |
|---|------|------|--------|-------|
| 2.7 | Architecture | `concepts/architecture.md` | NOT STARTED | Rust kernel, new event system |
| 2.8 | Modules | `concepts/modules.md` | NOT STARTED | 5 module types, updated contracts |
| 2.9 | Bundles | `concepts/bundles.md` | NOT STARTED | Thin bundle pattern, behaviors |
| 2.10 | Agents | `concepts/agents.md` | NOT STARTED | Context sinks, delegation patterns |
| 2.11 | Recipes | `concepts/recipes.md` | NOT STARTED | While loops, foreach, staged, approval gates |
| 2.12 | Skills | `concepts/skills.md` | NOT STARTED | Fork execution, power skills, slash commands |
| 2.13 | Hooks | `concepts/hooks.md` | NOT STARTED | Updated hook types and lifecycle |

### Tools Section (7 pages)

| # | Page | File | Status | Notes |
|---|------|------|--------|-------|
| 2.14 | Filesystem | `tools/filesystem.md` | NOT STARTED | apply_patch, @bundle paths |
| 2.15 | Bash | `tools/bash.md` | NOT STARTED | Background processes, timeouts |
| 2.16 | Search | `tools/search.md` | NOT STARTED | grep + glob updates |
| 2.17 | Web | `tools/web.md` | NOT STARTED | web_search + web_fetch |
| 2.18 | Task/Delegate | `tools/task.md` | NOT STARTED | Enhanced delegate (context_depth, context_scope) |
| 2.19 | LSP | `tools/lsp.md` | NOT STARTED | Updated operations |
| 2.20 | Recipes Tool | `tools/recipes-tool.md` | NOT STARTED | New operations (while, foreach, staged) |

### Bundles Section (4 existing pages)

| # | Page | File | Status | Notes |
|---|------|------|--------|-------|
| 2.21 | Foundation | `bundles/foundation.md` | NOT STARTED | Now 20+ agents (was 13), new patterns |
| 2.22 | Recipes | `bundles/recipes.md` | NOT STARTED | While loops, bash steps, enhanced features |
| 2.23 | LSP Python -> Python Dev | `bundles/lsp-python.md` | NOT STARTED | DEPRECATED: Redirect to python-dev bundle (see 3.4) |
| 2.24 | Design Intelligence | `bundles/design-intelligence.md` | NOT STARTED | Nine Dimensions, 7+ agents |

---

## Phase 3: New Bundle Documentation (8+ pages)

New bundles that need tutorial pages.

| # | Bundle | File | Status | Notes |
|---|--------|------|--------|-------|
| 3.1 | Superpowers | `bundles/superpowers.md` | NOT STARTED | TDD, modes, recipes, agents |
| 3.2 | DOT Graph | `bundles/dot-graph.md` | NOT STARTED | Graph authoring, analysis, discovery |
| 3.3 | Browser Tester | `bundles/browser-tester.md` | NOT STARTED | agent-browser, 3 agents |
| 3.4 | Python Dev | `bundles/python-dev.md` | NOT STARTED | python_check, code-intel |
| 3.5 | Team Knowledge | `bundles/team-knowledge.md` | NOT STARTED | Shared team memory |
| 3.6 | Dev Memory | `bundles/dev-memory.md` | NOT STARTED | Persistent cross-session memory |
| 3.7 | Projector | `bundles/projector.md` | NOT STARTED | Project orchestration |
| 3.8 | Stories | `bundles/stories.md` | NOT STARTED | Content generation from sessions |

---

## Phase 4: New Concepts (3+ pages)

New conceptual topics that don't exist yet.

| # | Concept | File | Status | Notes |
|---|---------|------|--------|-------|
| 4.1 | Modes | `concepts/modes.md` | NOT STARTED | Runtime behavior overlays |
| 4.2 | Model Routing | `concepts/routing.md` | NOT STARTED | Routing matrix, roles, fallback chains |
| 4.3 | Shadow Environments | `concepts/shadows.md` | NOT STARTED | OS-isolated testing sandboxes |

---

## Phase 5: New Tools Documentation (5+ pages)

New tools that need tutorial pages.

| # | Tool | File | Status | Notes |
|---|------|------|--------|-------|
| 5.1 | Python Check | `tools/python-check.md` | NOT STARTED | Ruff + Pyright quality checks |
| 5.2 | Todo | `tools/todo.md` | NOT STARTED | Task tracking tool |
| 5.3 | DOT Graph Tool | `tools/dot-graph.md` | NOT STARTED | Validate, render, analyze, prescan, assemble |
| 5.4 | Skills Tool | `tools/skills.md` | NOT STARTED | load_skill operations |
| 5.5 | Modes Tool | `tools/modes.md` | NOT STARTED | Mode management |
| 5.6 | Projector Tool | `tools/projector.md` | NOT STARTED | Project management operations |

---

## Phase 6: New Applications Section (4+ pages)

New section documenting Amplifier applications and UIs.

| # | Application | File | Status | Notes |
|---|-------------|------|--------|-------|
| 6.1 | Amplifier Chat | `apps/chat.md` | NOT STARTED | Browser-based conversational UI |
| 6.2 | Amplifier TUI | `apps/tui.md` | NOT STARTED | Rich terminal UI (Textual) |
| 6.3 | MCP Server | `apps/mcp-server.md` | NOT STARTED | VS Code / Copilot integration |
| 6.4 | Voice Bridge | `apps/voice-bridge.md` | NOT STARTED | Siri / CarPlay remote control |
| 6.5 | Apps Section Index | `apps/index.md` | NOT STARTED | Section overview |

---

## Phase 7: Complete Missing Sections

Pages that exist in nav but have no generated content.

### Skills Reference (3 pages -- nav entries need replacing)

NOTE: Scan revealed Playwright, Curl, Image Vision skills were REMOVED from ecosystem.
Replace with current skills. Update nav accordingly.

| # | Page | File | Status | Notes |
|---|------|------|--------|-------|
| 7.1 | Power Skills (/code-review, /mass-change) | `skills/power-skills.md` | NOT STARTED | Fork-execution skills |
| 7.2 | Skills Authoring | `skills/authoring.md` | NOT STARTED | Creating custom skills |
| 7.3 | Community Skills | `skills/community.md` | NOT STARTED | Available skill collections |

### Developer Setup (4 pages -- exist in nav, no content)

| # | Page | File | Status | Notes |
|---|------|------|--------|-------|
| 7.4 | CLI Tools | `dev-setup/cli-tools.md` | NOT STARTED | amplifier CLI reference |
| 7.5 | Shadow Workspace | `dev-setup/shadow-workspace.md` | NOT STARTED | Shadow environments |
| 7.6 | Remote Dev | `dev-setup/remote-dev.md` | NOT STARTED | Tailscale + Mosh + tmux |
| 7.7 | Debugging | `dev-setup/debugging.md` | NOT STARTED | Session debugging, traces |

### Advanced Topics (4 pages -- exist in nav, no content)

| # | Page | File | Status | Notes |
|---|------|------|--------|-------|
| 7.8 | Custom Bundle | `advanced/custom-bundle.md` | NOT STARTED | Building bundles from scratch |
| 7.9 | Custom Tool | `advanced/custom-tool.md` | NOT STARTED | Extending with new tools |
| 7.10 | Custom Recipe | `advanced/custom-recipe.md` | NOT STARTED | Building workflows |
| 7.11 | MCP Integration | `advanced/mcp-integration.md` | NOT STARTED | MCP server setup |

---

## Phase 8: Provider Documentation (NEW section)

| # | Provider | File | Status | Notes |
|---|----------|------|--------|-------|
| 8.1 | Anthropic | `providers/anthropic.md` | NOT STARTED | Claude models, API key setup |
| 8.2 | OpenAI | `providers/openai.md` | NOT STARTED | GPT models, configuration |
| 8.3 | Azure OpenAI | `providers/azure.md` | NOT STARTED | Enterprise deployment |
| 8.4 | Google Gemini | `providers/gemini.md` | NOT STARTED | Gemini models |
| 8.5 | Ollama | `providers/ollama.md` | NOT STARTED | Local/offline models |
| 8.6 | Providers Section Index | `providers/index.md` | NOT STARTED | Section overview |

---

## Phase 9: Site Structure & Navigation

| # | Task | Status | Notes |
|---|------|--------|-------|
| 9.1 | Update `mkdocs.yml` navigation | NOT STARTED | Add all new sections and pages |
| 9.2 | Update homepage (`content/index.md`) | NOT STARTED | New ecosystem overview, updated feature list |
| 9.3 | Regenerate all section index pages | NOT STARTED | Run `generate-index.yaml` per section |
| 9.4 | Update generation manifest | NOT STARTED | Track all generated content |

---

## Phase 10: Quality & Deploy

| # | Task | Status | Notes |
|---|------|--------|-------|
| 10.1 | Validate all internal links | NOT STARTED | Run link checker |
| 10.2 | Build site (`mkdocs build --clean`) | NOT STARTED | Verify clean build |
| 10.3 | Local preview and spot-check | NOT STARTED | `mkdocs serve` |
| 10.4 | Commit and push to GitHub Pages | NOT STARTED | Auto-deploys from `docs/` on master |

---

## Execution Notes

### Page Count Summary

| Category | Existing | New | Total |
|----------|----------|-----|-------|
| Quickstart | 6 | 0 | 6 |
| Concepts | 7 | 3 | 10 |
| Tools | 7 | 6 | 13 |
| Bundles | 4 | 8 | 12 |
| Skills | 0 (nav only) | 3 | 3 |
| Dev Setup | 0 (nav only) | 4 | 4 |
| Advanced | 0 (nav only) | 4 | 4 |
| Apps | 0 (new section) | 5 | 5 |
| Providers | 0 (new section) | 6 | 6 |
| **TOTAL** | **24** | **39** | **63** |

Plus homepage + section indexes = ~72 pages total.

### Recipe Strategy Per Phase

| Phase | Recipe | Agent Pattern |
|-------|--------|---------------|
| 0 | Manual fixes | Direct file-ops |
| 1 | `scan-ecosystem.yaml` | Automated |
| 2 | `generate-tool.yaml`, `generate-concept.yaml`, `generate-bundle.yaml`, `generate-quickstart.yaml` | Per-page subagent |
| 3 | `generate-bundle.yaml` | Per-page subagent |
| 4 | `generate-concept.yaml` | Per-page subagent |
| 5 | `generate-tool.yaml` or `generate-page.yaml` | Per-page subagent |
| 6 | `generate-page.yaml` or direct agent | Per-page subagent |
| 7 | Type-specific recipes | Per-page subagent |
| 8 | `generate-page.yaml` or direct agent | Per-page subagent |
| 9 | `generate-index.yaml` + manual | Mixed |
| 10 | Manual + bash | Direct |

### Parallelism

Content generation can be parallelized within a phase (3 concurrent subagents).
Phase ordering is: 0 -> 1 -> (2-8 can overlap) -> 9 -> 10.

---

## Progress Tracking

Update this section as work completes.

| Phase | Pages | Done | % |
|-------|-------|------|---|
| 0 - Infra | 6 tasks | 4 | 67% |
| 1 - Scan | 3 tasks | 2 | 67% |
| 2 - Refresh | 24 pages | 0 | 0% |
| 3 - New Bundles | 8 pages | 0 | 0% |
| 4 - New Concepts | 3 pages | 0 | 0% |
| 5 - New Tools | 6 pages | 0 | 0% |
| 6 - Apps | 5 pages | 0 | 0% |
| 7 - Missing | 11 pages | 0 | 0% |
| 8 - Providers | 6 pages | 0 | 0% |
| 9 - Navigation | 4 tasks | 0 | 0% |
| 10 - Deploy | 4 tasks | 0 | 0% |
| **TOTAL** | **~80 items** | **6** | **7.5%** |

### Session Log

| Date | Session | Work Completed |
|------|---------|----------------|
| 2026-04-02 | Initial planning | Phase 0 (infra fixes), Phase 1 (ecosystem scan), plan creation |