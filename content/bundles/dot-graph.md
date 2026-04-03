---
id: dot-graph
type: bundle
title: "DOT Graph Bundle"
---

# DOT Graph Bundle

## What is the DOT Graph Bundle?

Most tools treat diagrams as decoration -- something you draw after you understand a system. The DOT Graph bundle treats graphs as an **analysis substrate**. You can encode structure in a compact text format, render it visually, and query it programmatically -- all within the same artifact. A DOT file isn't just a picture; it's a data structure that both humans and machines can reason over.

The bundle provides three capabilities: **graph authoring** (create and refine DOT diagrams with expert knowledge), **graph analysis** (validate syntax, detect cycles, compute reachability, diff structural changes), and **codebase investigation** (the Parallax Discovery pipeline, which dispatches teams of specialized agents to map an unfamiliar repository's architecture automatically).

## What's Included

The bundle ships agents, a tool, skills, and a multi-agent discovery pipeline:

| Category | What It Provides |
|----------|-----------------|
| **Agents** | `dot-author`, `diagram-reviewer`, `discovery-orchestrator`, plus 8 pipeline agents |
| **Tool** | `dot_graph` -- validate, render, analyze, prescan, assemble |
| **Skills** | `dot-syntax`, `dot-patterns`, `dot-as-analysis`, `dot-quality`, `dot-graph-intelligence` |
| **Recipes** | Quick and deep discovery pipelines with approval gates |
| **Docs** | Full DOT language reference, pattern catalog, quality standards, Graphviz setup guide |

The shape vocabulary gives every node semantic meaning:

| Shape | Meaning | Use Case |
|-------|---------|----------|
| `box` | Process / component | Services, modules, pipeline steps |
| `ellipse` | Data / state | Data stores, states, inputs/outputs |
| `diamond` | Decision | Branch points, conditions |
| `cylinder` | Storage | Databases, file systems |
| `folder` | Group / namespace | Clusters, packages |
| `circle` / `doublecircle` | Start / terminal | FSM entry and exit states |

## Getting Started

Include the bundle in your Amplifier configuration:

```yaml
# .amplifier/config.yaml
bundles:
  - foundation
  - dot-graph
```

Once loaded, DOT capabilities are available immediately. Ask for a diagram, and Amplifier delegates to the right specialist.

> Create an architecture diagram for our authentication system.

`[Tool: dot-author]` Analyzes the request, selects the appropriate pattern (layered architecture), chooses shapes from the vocabulary (box for services, cylinder for the user store, diamond for the auth decision point), and produces complete, valid DOT with clusters, a legend, and graph-level attributes.

> Is this diagram production-ready?

`[Tool: diagram-reviewer]` Runs the 5-level review checklist (syntax, structure, quality, style, reconciliation) and returns a structured PASS/WARN/FAIL verdict with specific evidence for every finding.

## Key Agents

### dot-author

The expert DOT/Graphviz author. This agent carries the full DOT syntax reference, pattern library, and quality standards. It creates diagrams from scratch, refines existing DOT, converts natural language descriptions into graph form, and explains existing graph structures.

> Show our deployment pipeline as a graph.

`[Tool: dot-author]` Chooses the DAG/flowchart pattern, uses `rankdir=LR` for left-to-right workflow flow, applies the shape vocabulary (box for build steps, diamond for approval gates, cylinder for the artifact registry), and outputs complete DOT with a legend and consistent `snake_case` node IDs.

### diagram-reviewer

An independent quality reviewer -- deliberately separate from `dot-author` so no agent reviews its own work. It evaluates diagrams against a 5-level checklist:

1. **Syntax** -- valid DOT that parses correctly
2. **Structure** -- no orphan nodes, consistent flow direction, proper `cluster_` prefixes
3. **Quality** -- line count within targets, legend present when needed, `snake_case` IDs
4. **Style** -- shapes reflect semantic roles, edge styles are consistent, labels are clear
5. **Reconciliation** -- structural patterns that map to system problems (hub nodes with 10+ edges, isolated clusters, unlabeled cycles)

The verdict is always PASS, WARN, or FAIL, with every finding citing specific nodes, edges, or attributes. Level 5 reconciliation findings are especially valuable -- a hub node with too many edges isn't just a layout problem, it signals a missing abstraction layer in the system itself.

### discovery-orchestrator

The natural language entry point for codebase investigation. You don't invoke pipeline agents or recipes directly -- you describe what you want, and the orchestrator routes to the correct pipeline.

> Investigate this codebase and give me an architectural overview.

`[Tool: discovery-orchestrator]` Reads the fidelity signal ("overview" suggests quick), resolves the repository path, and invokes the quick discovery pipeline recipe. You see an approval gate with the selected investigation topics before agents are dispatched.

> Run a deep, thorough investigation of this repository.

`[Tool: discovery-orchestrator]` Detects the deep fidelity signal ("deep," "thorough"), and invokes the deep pipeline with full triplicate teams and reconciliation.

## Discovery Pipeline

The Parallax Discovery pipeline is the bundle's most distinctive capability. Named after astronomical parallax -- where depth is measured by observing from multiple positions -- it triangulates architectural understanding by examining code from three independent perspectives.

### The Three Perspectives

Each investigation topic gets a **triplicate team** of agents, each with fresh context and zero shared state:

| Agent | Perspective | What It Does |
|-------|------------|--------------|
| `code-tracer` | **HOW** | Traces execution paths using LSP, requires file:line evidence |
| `behavior-observer` | **WHAT** | Examines 10+ real instances, catalogs and quantifies patterns |
| `integration-mapper` | **WHERE/WHY** | Maps cross-boundary connections, finds composition effects |

Discrepancies between agents are the most valuable findings -- they mark where design intent and reality have separated.

### Fidelity Tiers

| Tier | Agents Per Topic | Use When |
|------|-----------------|----------|
| **quick** | 1 (code-tracer only) | Fast initial scan, time-constrained |
| **standard** | 2 (code-tracer + integration-mapper) | Default for most investigations |
| **deep** | 3 (full triplicate team) | High-stakes, unfamiliar, or complex systems |

### Pipeline Stages

The pipeline runs in three stages with an approval gate after the first:

**Stage 1 -- Scan.** The `discovery-prescan` agent scans the repository structure (using the `dot_graph` tool's prescan operation) and selects 3-7 investigation topics based on fidelity. You review and approve the topics before agents are dispatched.

**Stage 2 -- Investigate.** Each topic dispatches its agent team in parallel. Agents write to isolated subdirectories -- no shared mutable state.

**Stage 3 -- Synthesize.** The `discovery-synthesizer` reconciles per-topic agent findings into consensus diagrams. The `discovery-combiner` merges module-level results. The `discovery-level-synthesizer` and `discovery-overview-synthesizer` build progressively higher-level views until a final `overview.dot` and `overview.md` capture the system architecture.

### Pipeline Agents

Beyond the three investigator roles, the pipeline includes synthesis agents:

| Agent | Role |
|-------|------|
| `discovery-prescan` | Selects investigation topics from structural inventory |
| `discovery-synthesizer` | Reconciles multi-agent findings into consensus DOT |
| `discovery-combiner` | Merges module-level results across topics |
| `discovery-level-synthesizer` | Builds subsystem-level views from module findings |
| `discovery-overview-synthesizer` | Produces the final system-spine overview |
| `discovery-architecture-writer` | Writes the architecture overview document |

## Practical Example: Investigating a Codebase

Here's what a full discovery run looks like end-to-end. You've just cloned an unfamiliar repository and need to understand its architecture.

**Step 1 -- Ask**

> Investigate this codebase and map the architecture.

`[Tool: discovery-orchestrator]` Resolves the repo path, selects the quick pipeline (no deep signal), and invokes the recipe.

**Step 2 -- Approve topics**

The pipeline scans the repository and presents its selected topics for approval:

```
Quick Discovery Pipeline -- Scan Complete

Repository:  /home/user/project
Fidelity:    standard

Selected investigation topics (4 topics):
  * API Layer (api-layer)
    HTTP routing, middleware chain, and request lifecycle
  * Data Model (data-model)
    Schema definitions, ORM mappings, and migration patterns
  * Auth Subsystem (auth-subsystem)
    Authentication flow, session management, and token handling
  * Event Pipeline (event-pipeline)
    Async event dispatch, consumer registration, and retry logic

Type APPROVE to proceed with investigation, or DENY to cancel.
```

**Step 3 -- Agents investigate**

After approval, agent teams dispatch in parallel. Each topic gets its assigned agents (2 for standard fidelity), writing findings and diagrams to isolated directories under `.discovery/modules/`.

**Step 4 -- Synthesis**

The synthesizer agents reconcile findings, build consensus diagrams, and the `dot-author` produces a final system-spine `overview.dot` with 10-15 nodes representing the primary components and their relationships.

**Step 5 -- Results**

You receive an `overview.md` summarizing the architecture and an `overview.dot` you can render, query, or diff against future runs:

```
.discovery/
  output/
    overview.md          # Architecture summary
    overview.dot         # System-spine diagram
    modules/
      api-layer/diagram.dot
      data-model/diagram.dot
      ...
  last-run.json          # Change detection for incremental updates
```

The next time you run discovery, change detection compares git commits and only re-investigates what changed.

## DOT as Analysis Substrate

The `dot_graph` tool goes beyond rendering. Its analysis operations let you query graph structure programmatically:

> Analyze the architecture diagram for cycles.

`[Tool: dot_graph]` Runs the `analyze` operation with `cycles` analysis -- returns any circular dependencies found in the graph structure.

> Which components are reachable from the API gateway?

`[Tool: dot_graph]` Runs `reachability` analysis from the gateway node -- returns every component in the dependency chain.

> What changed between the last two discovery runs?

`[Tool: dot_graph]` Runs `diff` analysis comparing two DOT files -- reports added nodes, removed edges, and structural changes.

The full analysis toolkit:

| Operation | What It Does |
|-----------|--------------|
| `stats` | Node count, edge count, density, DAG detection, components |
| `reachability` | All nodes reachable from a source |
| `cycles` | Detect circular dependencies |
| `paths` | All simple paths between two nodes |
| `critical_path` | Longest path in a DAG |
| `diff` | Structural differences between two graphs |

This is what makes DOT an analysis substrate rather than just a visual format -- the same artifact serves humans and machines.

## Tips

**Use discovery-orchestrator, not the pipeline directly.** The orchestrator handles fidelity selection, repo path resolution, and recipe invocation. Invoking pipeline agents or recipes directly bypasses its routing logic.

**Start with quick or standard fidelity.** Deep investigations dispatch three agents per topic and take significantly longer. Use deep when the stakes justify it -- unfamiliar codebases, pre-refactor audits, or architecture reviews.

**Let diagram-reviewer check dot-author's work.** No agent should review its own output. The author-reviewer separation is deliberate -- always run `diagram-reviewer` after `dot-author` for objective quality assessment.

**Treat reconciliation findings as system signals.** When `diagram-reviewer` flags a hub node with 10+ edges, that's not a diagram problem -- it's a design problem. Structural patterns in graphs map directly to architectural issues in code.

**Use incremental discovery.** The pipeline writes `last-run.json` with the git commit hash. Subsequent runs detect changes and only re-investigate affected topics.

## Next Steps

- Learn about the [Foundation Bundle](./foundation.md) for core development agents
- Explore the [Recipes Bundle](./recipes.md) for multi-step workflow orchestration
- See [Design Intelligence Bundle](./design-intelligence.md) for UI/UX specialized agents
- Read about [Agents](../concepts/agents.md) for how agent delegation works under the hood
