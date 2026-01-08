# Architecture Explanation from Lead Dev (bkrabach)

**Source:** Direct communication from Brian Krabach (lead dev)
**Date:** January 2026
**Status:** [INCORPORATED] - Added to concepts/architecture.md on 2026-01-08

---

## Original Text

There are multiple ways to infuse capabilities into a system like this.

### Agents

Agents are "just" bundles that we spawn a sub-session and the main session drives as the "user" instead of us - it's like giving Amplifier its own instances of Amplifier to drive, and each "agent" bundle is just a different, specialized bundle ideally focused on specific tasks (which is why it's our "task" tool that they use to spawn these, and the intent, call them for a "task" to be done).

### Tools

Tools are code that can be called by the LLMs that get called via the orchestrators. If the LLM decides to call a tool instead of generating a response, the LLM response will include a specific tool call request with parameters and then this is executed (no models used unless the tool does under its hood) and then the results are passed back to the LLM to see what it wants to do next - this continues until the LLM stops asking for tools to be called.

### Hooks

Hooks are code that listens to lifecycle events in the orchestrator (though initiated frequently by other modules - tools, other hooks, etc. - as well) and then trigger code (that can call models if the code/hook feature requires such), so NO LLM making decisions IF they should be called and then it starts code-first, so full control.

### Orchestrators

Orchestrators are the main engine that drives your "Amplifier Session" and can make the experience radically different - it is code-first that connects and leverages as much of the above stuff as it is coded to do. Our main ones are agentic-loop orchestrators, but we could have a recipe-runner as an orchestrator, or Paul has one he's building that is an observer-pattern orchestrator where it has other instances "observing" a very limited capability (no tools, etc.) main session and then the observers do work and push context in instead of the other way around from how we do today. Lots of other ways you could do very powerful things through this module slot in a session.

### Context Modules

Context modules are the managers of your context, storing/loading/"compacting" your history, can be the place to connect or implement "memory" systems, etc.

### Providers

Providers are another module type and are generally vendor level and support many/all of their models, but could also be 1:1 provider:model approach as well, or even some aggregator-provider w/ many models from many other providers - lots of room here too.

### Bundles

So then bundles... this is our "packaging" - it lets us group together the various parts (not just the above but other context files, etc.). It could be as lightweight as a bundle around a tool module or as fully featured (like the foundation bundle) to be a "ready to run", everything included bundle. Bundles can include other bundles, which compose on top of each other - so that is how the foundation bundle works... it's not actually ALL in the foundation bundle, but it is our opinionated "this is what we use" bundle that has included other smaller bundles (we call them "behavior bundles" by convention, there is no code for "behavior" concept, it's just convention).

### Skills

So skills... Skills is "just" a bundle. It has some tools, context files, agent(s), etc. (a common composition of a behavior bundle). In this case, it's a bundle that can load "skills" which is NOT an Amplifier concept, but this is a way we extend Amplifier for things like this. Marc has done others this way such as MCP, Anthropic plugins, etc. In fact, we will likely be moving all agent-related stuff into the task bundle (or more likely naming it the agent bundle and the task tool is part of it) to follow this same pattern since technically agents are not actually a "core" concept, but is something we brought in on top of the core through the foundation lib.

---

## Key Insights for Tutorial

1. **Agents are bundles** - Not a special concept, just bundles spawned as sub-sessions
2. **Tools are LLM-driven** - Model decides when to call them
3. **Hooks are code-driven** - No LLM decision, pure event response
4. **Orchestrators are the engine** - Radically different behaviors possible
5. **Bundles are packaging** - Composition via includes
6. **Skills extend via bundles** - Not core, but a pattern we use

## Suggested Placement

Create "Architecture Deep Dive" page in Core Concepts that synthesizes this with the existing module explanations.
