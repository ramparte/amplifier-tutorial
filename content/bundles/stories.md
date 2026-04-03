---
id: stories
type: bundle
title: "Stories Bundle"
---

# Stories Bundle

## What is the Stories Bundle?

You just finished an extraordinary session. Over two hours, Amplifier helped you build a new feature from scratch -- researching the problem, writing the code, fixing edge cases, deploying it. The session log is sitting in `events.jsonl`, full of breakthrough moments and clever problem-solving. But nobody will ever know, because session logs are unreadable and you don't have time to write it up.

The Stories bundle turns Amplifier's own activity into polished content. It mines git repositories, session files, and development history to find stories worth telling -- then generates case studies, blog posts, release notes, executive summaries, changelogs, migration guides, and presentation decks. You say "make an Amplifier story about the new auth feature," and the bundle researches the git history, designs a narrative arc, and produces a self-contained HTML deck you can share immediately.

This isn't a generic content generator. Every agent in the bundle is a specialist -- one researches, one strategizes, one writes technical docs, another writes for executives. The storyteller agent produces "Useful Apple Keynote" style HTML presentations with dark backgrounds and high-density layouts. The release-manager reads git tags and generates changelogs automatically. The whole pipeline is evidence-backed: every metric in every output traces to actual data from your repos and sessions.

## What's Included

The bundle provides eleven specialist agents and four automated recipes:

| Agent | Role |
|-------|------|
| **story-researcher** | Mines git repos, sessions, and ecosystem activity for stories worth telling |
| **content-strategist** | Determines what stories to tell and how to frame them |
| **case-study-writer** | Turns sessions into structured case study narratives |
| **technical-writer** | Deep technical documentation and guides |
| **marketing-writer** | External communication and feature announcements |
| **executive-briefer** | High-level summaries for decision-makers |
| **community-manager** | Community engagement content |
| **content-adapter** | Converts between formats and audiences |
| **data-analyst** | Transforms raw data into insights and metrics |
| **release-manager** | Automated changelogs, migration guides, and release announcements from git tags |
| **storyteller** | Creates polished HTML presentation decks |

And four recipes for automated workflows:

| Recipe | What It Does |
|--------|-------------|
| **session-to-case-study** | Transforms a session's `events.jsonl` into a professional case study |
| **git-tag-to-changelog** | Generates semantic changelogs from git tag history |
| **weekly-digest** | Produces regular ecosystem updates with zero manual work |
| **blog-post-generator** | Creates feature stories for community distribution |

## Getting Started

The simplest way to start is to ask for a presentation about something you've built.

> Tell a story about the new shadow environments feature

```
[Tool: task] Delegating to stories:story-researcher...
  Mining git history for: shadow environments
  Scanning 3 repos...
  Found: 47 commits, 12 PRs, 3 contributors
  Timeline: Jan 8 - Jan 22, 2026 (14 days)
  Research complete. Returning structured data.

[Tool: task] Delegating to stories:storyteller...
  Designing narrative arc: problem → solution → impact → velocity
  Building HTML deck: "Useful Apple Keynote" style
  Saved: docs/shadow-environments.html
  Opening in browser...
```

The storyteller always starts with research. It delegates to `story-researcher` first, which gathers evidence from git history, PRs, and commit patterns. Only then does it design and build the deck -- and every number in the presentation traces back to the research output.

> Make that into an executive summary instead

```
[Tool: task] Delegating to stories:executive-briefer...
  Adapting shadow environments story for executive audience
  Key metrics: 14-day development cycle, 3x faster environment provisioning
  Output: executive-summary-shadow-environments.md
```

Same research, different audience. The executive-briefer strips technical detail and leads with business impact.

## Key Agents

### story-researcher: Finding Stories in Your Data

The researcher is the foundation of the pipeline. It mines data sources that other agents consume:

> What stories can we tell from the last month of development?

```
[Tool: task] Delegating to stories:story-researcher...
  Scanning git activity across 5 repos...
  Analyzing session files from last 30 days...

  Stories worth telling:
  1. Recipe cancellation feature — 3 days, 1 developer, 23 files changed
  2. LSP integration rewrite — significant velocity spike in week 2
  3. Cross-session memory breakthrough — novel architecture, high impact
  4. Bundle composition redesign — 4 contributors, breaking change handled cleanly

  Recommendation: #3 has the strongest narrative arc.
```

The researcher looks for signals: velocity spikes, breakthrough moments in sessions, multi-contributor collaboration, and clean handling of complex problems. It reports what it finds and what it can't find -- it never fabricates metrics.

### storyteller: Polished HTML Decks

The storyteller creates self-contained HTML presentations in a high-density "Useful Apple Keynote" style -- dark backgrounds, blue gradients, and information-rich slides designed for engineers.

> Create a deck about the recipe cancellation feature

```
[Tool: task] Delegating to stories:story-researcher...
  Research complete. 23 commits, 2 PRs, 1 contributor.

[Tool: task] Delegating to stories:storyteller...
  Narrative: problem (stuck recipes) → solution (graceful cancellation)
             → impact (zero orphaned processes) → velocity (3 days)
  Creating 12-slide HTML deck...
  Antagonistic review: checking all metrics against research...
  ✓ All numbers verified. Sources slide included.
  Saved: docs/recipe-cancellation.html
```

Every deck goes through an antagonistic review before saving. The storyteller checks each number, date, and impact claim against the research data. If the research said "~45 commits," the deck says "~45 commits" -- not "45 commits" and certainly not "50 commits."

### release-manager: Changelogs from Git Tags

The release-manager automates release documentation. Point it at a git tag and it generates everything:

> Generate release notes for the v2.0.0 tag

```
[Tool: task] Delegating to stories:release-manager...
  Analyzing commits between v1.9.0 and v2.0.0...
  Found: 8 features, 3 breaking changes, 12 fixes

  Generated:
  - CHANGELOG.md (semantic versioning format)
  - migration-guide-v2.md (breaking change details)
  - release-announcement.md (community-ready)
```

The release-manager follows Keep a Changelog format and semantic versioning conventions. When it finds breaking changes, it automatically generates a migration guide explaining what changed and how to update.

### content-adapter: One Story, Many Formats

The content-adapter transforms existing content between formats and audiences:

> Take the shadow environments case study and make it a blog post

```
[Tool: task] Delegating to stories:content-adapter...
  Source: case-studies/shadow-environments.md (technical, 2500 words)
  Target: blog post (community audience, 800 words)
  Adapting tone, stripping implementation details, adding context...
  Output: blog-shadow-environments.md
```

## Recipes: Automated Content Pipelines

### session-to-case-study

The signature recipe. It takes a raw session file and produces a professional case study:

> Run the session-to-case-study recipe with session_file=~/.amplifier/sessions/2026-01-15/events.jsonl

```
[Tool: recipes]
  operation: execute
  recipe_path: "@stories:recipes/session-to-case-study.yaml"
  context: { session_file: "~/.amplifier/sessions/2026-01-15/events.jsonl" }

  Step 1/4: research — Analyzing session events...
    Session: 2h 15m, 47 tool calls, 3 agents used
    Key moments: auth middleware breakthrough at minute 45
  Step 2/4: strategize — Designing narrative...
    Angle: "From blank file to production auth in one session"
  Step 3/4: write — Generating case study...
  Step 4/4: review — Checking claims against evidence...
  ✓ Case study saved: case-studies/auth-middleware-session.md
```

The recipe runs four phases: research extracts raw data, a strategist picks the narrative angle, a writer produces the document, and a reviewer verifies claims. Each phase is a separate agent with its own expertise.

### git-tag-to-changelog

Automates the tedious work of writing release notes:

> Run the changelog recipe for all tags since v1.5.0

```
[Tool: recipes]
  operation: execute
  recipe_path: "@stories:recipes/git-tag-to-changelog.yaml"
  context: { since_tag: "v1.5.0" }

  Processing 4 releases: v1.5.0 → v1.6.0 → v1.7.0 → v1.8.0 → v2.0.0
  Generated CHANGELOG.md with 4 release sections.
```

## Practical Example: "Make an Amplifier Story"

Here's the full workflow for creating a story about Amplifier itself -- the kind of content that explains what you've built and why it matters.

> Make an Amplifier story about how we built cross-session memory

```
[Tool: task] Delegating to stories:story-researcher...
  Mining amplifier-core and dev-memory repos...
  Found: 89 commits over 3 weeks, 2 contributors
  Key insight: architecture shifted from vector DB to plain YAML
  Session data: 4 sessions with memory-related breakthroughs

[Tool: task] Delegating to stories:content-strategist...
  Story angle: "Why we chose simplicity over sophistication"
  Target audiences: engineers (primary), leadership (secondary)
  Recommended formats: HTML deck + executive summary

[Tool: task] Delegating to stories:storyteller...
  Building 15-slide deck...
  Narrative: complex problem → simple solution → surprising benefits
  Antagonistic review: ✓ all 12 metrics verified
  Saved: docs/cross-session-memory.html

[Tool: task] Delegating to stories:executive-briefer...
  One-page summary: "Persistent memory across sessions,
    plain-text storage, zero external dependencies"
  Saved: executive-summary-cross-session-memory.md
```

One prompt produced a presentation deck and an executive summary, both grounded in real git data and session evidence.

## Tips

**Always let the researcher run.** The storyteller mandates research before creating any deck. Don't skip it or try to provide metrics manually -- the pipeline is designed around evidence-backed claims, and the antagonistic review will flag anything that doesn't trace to research.

**Use recipes for repeatable workflows.** If you're generating case studies or changelogs regularly, use the recipes instead of ad-hoc prompts. Recipes run the full pipeline every time -- research, strategy, writing, review -- so nothing gets skipped.

**Match the agent to the audience.** Technical-writer for engineers, executive-briefer for leadership, marketing-writer for external announcements, community-manager for developer communities. Each agent knows its audience's expectations and adjusts tone, detail level, and framing accordingly.

**Check the generated HTML locally.** The storyteller auto-opens decks in your browser after creation. Review before deploying -- the antagonistic review catches metric errors, but narrative tone and emphasis are judgment calls that benefit from a human eye.

**Deploy with the included script.** The bundle includes `deploy.sh` for pushing decks to SharePoint. Run `./deploy.sh docs/my-deck.html` for a single deck or `./deploy.sh` for all of them.

**Convert to PowerPoint when needed.** For audiences that need `.pptx`, use the included converter: `uv run --with python-pptx --with beautifulsoup4 tools/html2pptx_v2.py docs/deck.html output.pptx`. It handles card grids, code blocks, and merged headers natively.

## Next Steps

- Learn about the [Foundation Bundle](./foundation.md) for core Amplifier capabilities
- Explore [Recipes](../concepts/recipes.md) to understand how multi-step pipelines work
- See [Agents](../concepts/agents.md) for how specialist agents are defined and delegated to
- Read about the [Projector Bundle](./projector.md) for tracking the projects you're telling stories about
