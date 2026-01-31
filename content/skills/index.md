---
id: skills-index
type: section-index
title: Skills Guide
---

# Skills Guide

Skills are loadable knowledge modules that provide domain expertise on demand. Unlike always-present context, skills are loaded when needed, keeping the base context lean. This section covers how to use existing skills, create custom ones, and integrate them into your workflows.

## Section Contents

| Page | Description |
|------|-------------|
| [Curl](./curl.md) | HTTP requests and API interactions |
| [Image Vision](./image-vision.md) | Image understanding and analysis |
| [Playwright](./playwright.md) | Browser automation and testing |

## Quick Tips

- **Load on demand** - Only load skills when the domain knowledge is needed
- **Check before loading** - Use `info` operation to preview skill metadata
- **Skills complement docs** - They provide workflows, not just reference
- **Workspace priority** - Project skills override user skills
- **Companion files** - Skills can reference additional resources

## Skills vs Context

| Aspect | Skills | Context |
|--------|--------|---------|
| Loading | On-demand | Always present |
| Scope | Domain-specific | General behavior |
| Size | Can be large | Should be concise |
| Location | Discoverable directories | Bundle-defined |

## Where to Start

**Want HTTP capabilities?** Start with [Curl](./curl.md) for HTTP requests and API interactions.

**Working with images?** Jump to [Image Vision](./image-vision.md) for image understanding.

**Need browser automation?** See [Playwright](./playwright.md) for browser testing.

## Skill Operations

```python
# List all available skills
load_skill(list=True)

# Search for skills by topic
load_skill(search="python")

# Check skill metadata
load_skill(info="skill-name")

# Load full skill content
load_skill(skill_name="skill-name")
```

## Skill Discovery Paths

Skills are discovered from configured directories in priority order:

1. **Workspace skills** - `.amplifier/skills/` (project-specific)
2. **User skills** - `~/.amplifier/skills/` (personal)
3. **Bundle skills** - From active bundles
4. **Collection skills** - From installed collections

First match wins—workspace skills override user skills.

## Next Steps

After understanding skills, explore [Advanced](../advanced/index.md) topics for complex patterns like recipes and multi-agent workflows.
