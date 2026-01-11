---
id: skills-index
type: section-index
title: Skills Reference
---

# Skills Reference

Skills are packages of domain knowledge that agents can load on demand. Unlike context files that are always present, skills provide specialized expertise that's loaded only when relevant.

This section covers skill creation, discovery, and effective use patterns.

## Section Contents

| Page | Description |
|------|-------------|
| [Skill Basics](./basics.md) | What skills are and how they work |
| [Built-in Skills](./built-in.md) | Available skills in the foundation bundle |
| [Creating Skills](./creating.md) | Build your own domain-specific skills |
| [Skill Structure](./structure.md) | Anatomy of a skill package |
| [Discovery](./discovery.md) | How agents find and load skills |
| [Companion Files](./companion-files.md) | Including examples and templates |
| [Best Practices](./best-practices.md) | Patterns for effective skills |
| [Publishing](./publishing.md) | Sharing skills with the community |

## Quick Tips

- **Skills are lazy-loaded** — They only consume context when actually needed
- **One domain per skill** — Keep skills focused on a single area of expertise
- **Include examples** — Companion files with working code help agents apply knowledge
- **Version carefully** — Skills become dependencies; changes affect users
- **Search first** — Use `load_skill(search="term")` to find relevant skills

## Skill Architecture

```
skills/
└── my-skill/
    ├── skill.yaml       # Skill manifest (name, description, version)
    ├── skill.md         # Main knowledge content
    └── examples/        # Companion files
        ├── basic.py
        └── advanced.py
```

## Skill Operations

| Operation | Description | Example |
|-----------|-------------|---------|
| List | Show available skills | `load_skill(list=True)` |
| Search | Find by name/description | `load_skill(search="python")` |
| Info | Get metadata only | `load_skill(info="curl")` |
| Load | Load full content | `load_skill(skill_name="curl")` |

## Where to Start

**New to skills?** Begin with [Skill Basics](./basics.md) to understand the skill system and when to use it.

**Want to create skills?** Follow [Creating Skills](./creating.md) for a hands-on guide to building your first skill.

**Looking for existing skills?** Check [Built-in Skills](./built-in.md) for what's already available.

## Built-in Skills Overview

| Skill | Domain | Use Case |
|-------|--------|----------|
| `curl` | HTTP/APIs | Testing REST endpoints |
| `playwright` | Browser | Web automation and scraping |
| `image-vision` | Vision | Analyzing images with LLMs |

## Skill vs Context

| Aspect | Skills | Context Files |
|--------|--------|---------------|
| Loading | On-demand | Always present |
| Scope | Specialized domain | General guidance |
| Size | Can be large | Should be concise |
| Updates | Versioned | Part of bundle |

## Common Patterns

```yaml
# In an agent's workflow:
1. Recognize domain-specific task
2. Search for relevant skill
3. Load skill into context
4. Apply skill knowledge
5. Reference companion files if needed
```

## Related Sections

- [Quickstart: Creating Skills](../quickstart/creating-skills.md)
- [Bundles: Context Files](../bundles/context-files.md)
- [Advanced: Skill Development](../advanced/skill-development.md)
