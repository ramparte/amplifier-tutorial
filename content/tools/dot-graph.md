---
id: tool-dot-graph
type: tool
title: "DOT Graph Tool"
---

# DOT Graph Tool

Code is structure — modules depend on modules, functions call functions, data flows through pipelines. But staring at code doesn't always reveal the shape of that structure. The DOT Graph tool lets you express structure as diagrams in Graphviz DOT format and then *do things with it*: validate that your graph syntax is correct, render it to an image, analyze its properties without rendering, scan a codebase into a structural inventory, and assemble module-level graphs into system-level overviews.

This isn't just a rendering tool. The analysis operations let you ask structural questions — are there cycles in this dependency graph? What's the critical path? What changed between two versions of an architecture? — and get precise, computed answers instead of eyeballing a picture.

## Core Capabilities

The DOT Graph tool has five operations, each serving a different stage of working with graphs.

### Validate — Check Before You Render

Validation runs your DOT content through up to three layers of checking:

| Layer | What It Checks | Catches |
|-------|---------------|---------|
| **syntax** | DOT grammar is well-formed | Missing semicolons, unclosed braces, bad attribute syntax |
| **structural** | Graph makes logical sense | Duplicate edges, orphaned nodes, empty subgraphs |
| **render** | Graphviz can actually produce output | Layout engine errors, unsupported attributes |

By default, all three layers run. You can also request specific layers if you only care about one:

> Validate this architecture diagram

```
[Tool: dot_graph] validate
  dot_content: "digraph arch { frontend -> api; api -> database; api -> cache; }"
  Syntax: OK
  Structural: OK
  Render: OK — valid DOT, ready to render
```

When something is wrong, the tool tells you exactly where:

> Check this graph for errors

```
[Tool: dot_graph] validate
  dot_content: "digraph broken { a -> b; b -> c; c -> }"
  Syntax: ERROR — expected node identifier at position 42, found "}"
```

Validate early and validate often. Catching a syntax error before rendering saves a round trip.

### Render — From Text to Image

Rendering takes DOT content and produces a visual output file. The tool supports multiple formats and layout engines:

**Formats:** SVG, PNG, PDF, PS, EPS, JSON

**Layout engines:**

| Engine | Best For |
|--------|----------|
| `dot` | Directed graphs, hierarchies, flowcharts (default) |
| `neato` | Undirected graphs, network topologies |
| `fdp` | Large undirected graphs with clustering |
| `sfdp` | Very large graphs (thousands of nodes) |
| `twopi` | Radial layouts from a central node |
| `circo` | Circular layouts, ring structures |

> Render the service architecture as a PNG

```
[Tool: dot_graph] render
  dot_content: "digraph services {
    rankdir=LR;
    gateway -> auth; gateway -> api; api -> db;
    api -> cache; auth -> db;
  }"
  format: "png"
  engine: "dot"
  output_path: "/tmp/services.png"

Rendered to /tmp/services.png (12KB)
```

You can also let the tool auto-generate a temp file path if you don't care where it lands:

> Quick render of this graph to SVG

```
[Tool: dot_graph] render
  dot_content: "digraph { a -> b -> c -> d }"
  format: "svg"

Rendered to /tmp/dot_graph_a3f2.svg (4KB)
```

### Analyze — Structure Without Pixels

This is where the DOT Graph tool goes beyond visualization. The analyze operations compute structural properties of your graph using networkx — no rendering needed, no images produced. Just answers.

**stats** — basic graph metrics:

```
[Tool: dot_graph] analyze
  analysis: "stats"
  dot_content: "digraph { a -> b; b -> c; c -> d; a -> d; }"

  Nodes: 4
  Edges: 4
  Density: 0.33
  Is DAG: yes
  Connected components: 1
```

**cycles** — detect circular dependencies:

> Are there any cycles in the module dependency graph?

```
[Tool: dot_graph] analyze
  analysis: "cycles"
  dot_content: "digraph deps {
    auth -> users; users -> permissions;
    permissions -> auth; api -> auth;
  }"

  Cycles found: 1
    auth → users → permissions → auth
```

That cycle means `auth`, `users`, and `permissions` are mutually dependent — a common design smell that's hard to spot by reading imports.

**reachability** — what can be reached from a given node:

```
[Tool: dot_graph] analyze
  analysis: "reachability"
  source_node: "gateway"

  Reachable from gateway: auth, api, db, cache (4 nodes)
```

**paths** — all simple paths between two nodes:

```
[Tool: dot_graph] analyze
  analysis: "paths"
  source_node: "gateway"
  target_node: "db"

  Paths (2 found):
    gateway → api → db
    gateway → auth → db
```

**critical_path** — the longest path in a DAG, showing the bottleneck chain:

```
[Tool: dot_graph] analyze
  analysis: "critical_path"

  Critical path: gateway → api → processor → validator → db (length: 4)
```

**diff** — structural differences between two versions of a graph:

```
[Tool: dot_graph] analyze
  analysis: "diff"
  dot_content: "digraph v1 { a -> b; b -> c; }"
  dot_content_b: "digraph v2 { a -> b; b -> c; c -> d; b -> d; }"

  Added nodes: d
  Added edges: c → d, b → d
  Removed nodes: (none)
  Removed edges: (none)
```

**subgraph_extract** — pull a named cluster out as a standalone graph:

```
[Tool: dot_graph] analyze
  analysis: "subgraph_extract"
  cluster_name: "cluster_auth"

  Extracted cluster_auth as standalone DOT (3 nodes, 4 edges)
```

### Prescan — Inventory Your Codebase

Before you can diagram a system, you need to know what's in it. Prescan walks a repository and produces a structural inventory — languages detected, modules found, files per module:

> Scan this repo to see what we're working with

```
[Tool: dot_graph] prescan
  repo_path: "/home/user/project"

  Languages: Python (87%), JavaScript (10%), YAML (3%)
  Modules: 12
    src/auth/      — 8 files, 1200 lines
    src/api/       — 14 files, 2800 lines
    src/models/    — 6 files, 900 lines
    ...
```

Prescan output feeds directly into assembly — once you know the modules, you can diagram each one and merge them together.

### Assemble — Build the Big Picture

Assembly takes per-module DOT files and merges them into subsystem and overview graphs. This is hierarchical: module-level diagrams combine into subsystem diagrams, which combine into a full system overview.

> Assemble the architecture diagrams from per-module DOTs

```
[Tool: dot_graph] assemble
  manifest: { ... module DOT mappings ... }
  output_dir: "/tmp/architecture"

  Assembled:
    /tmp/architecture/subsystem_backend.dot
    /tmp/architecture/subsystem_frontend.dot
    /tmp/architecture/overview.dot
  Rendered PNGs alongside each DOT file.
```

By default, assembly also renders each assembled graph to PNG. Set `render_png: false` if you only want the DOT files.

Assembly supports incremental updates — pass `invalidated_modules` to re-assemble only the subsystems affected by changes, rather than rebuilding everything.

## Tips

**Validate before rendering.** A validation call is cheap and catches syntax errors instantly. A failed render gives you less helpful error messages and wastes more time.

**Use analysis for questions, render for communication.** If you want to know whether cycles exist, use `analyze`. If you want to show the architecture to your team, use `render`. They serve different purposes.

**Pick the right layout engine.** Hierarchical code flows want `dot`. Network topologies want `neato`. If your graph looks tangled with the default engine, try a different one before restructuring the DOT.

**Diff before and after refactors.** Diagram the dependency structure before a refactor, then again after. The `diff` analysis shows exactly what changed structurally, which is much clearer than diffing code.

**Leverage incremental assembly.** For large projects, don't reassemble everything when one module changes. Use `invalidated_modules` to rebuild just the affected subsystems.

## Next Steps

- Learn about the [Bash Tool](./bash.md) for running Graphviz commands directly when you need lower-level control
- See [Filesystem Tools](./filesystem.md) for reading and writing DOT files
- Explore [Delegate Tool](./task.md) for delegating complex architecture analysis to sub-agents
