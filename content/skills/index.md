---
id: skills-overview
type: skills
title: "Skills Reference"
---

# Skills Reference

Skills are reusable knowledge packages that give Amplifier domain expertise.

## What Are Skills?

Skills follow the [Anthropic Skills format](https://github.com/anthropics/skills) - structured knowledge that can be loaded on demand.

A skill provides:
- **Workflows** - Decision trees for complex tasks
- **Patterns** - Best practices and examples
- **Troubleshooting** - Common errors and solutions
- **Setup guides** - Installation and configuration

## Available Skills

| Skill | Purpose | Guide |
|-------|---------|-------|
| [Playwright](playwright.md) | Browser automation | UI testing, scraping |
| [Curl](curl.md) | HTTP client | API testing, debugging |
| [Image Vision](image-vision.md) | Image analysis | OCR, visual understanding |

## Loading Skills

```bash
# In a session
amplifier> Load the playwright skill

# Or explicitly
amplifier> /skill load playwright
```

## Skills vs Tools

| Aspect | Skill | Tool |
|--------|-------|------|
| **What it is** | Knowledge | Capability |
| **How it helps** | Teaches approach | Does work |
| **When loaded** | On demand | Always available |
| **Example** | "How to use Playwright" | "Run this bash command" |

Skills teach *how to think*. Tools *do things*.

## Progressive Loading

Skills load efficiently:

| Level | Content | Tokens |
|-------|---------|--------|
| **Metadata** | Name, description | ~100 |
| **Core** | Main SKILL.md | ~1,000 |
| **Reference** | Patterns, setup | On demand |

Only what's needed is loaded.

## Skill Structure

```
my-skill/
├── SKILL.md           # Required: Core workflow
├── patterns.md        # Optional: Advanced patterns
├── setup.md           # Optional: Installation
└── troubleshooting.md # Optional: Common issues
```

## Quick Comparison

| Task | Skill to Load |
|------|---------------|
| Browser automation | Playwright |
| API testing | Curl |
| Image analysis | Image Vision |

## Using Skills Effectively

### Load Before Task

```
> Load the playwright skill
> Now test the login form
```

### Combine Skills

```
> Load curl and image-vision skills
> Test the API and verify the returned image
```

### Check What's Available

```
> What skills are available?
> /skills list
```

## Creating Custom Skills

See [Anthropic Skills format](https://github.com/anthropics/skills) for the specification.

Basic structure:

```markdown
# Skill Name

Brief description.

## When to Use

- Scenario 1
- Scenario 2

## Workflow

Decision tree...

## Core Patterns

### Pattern 1
[Example]

### Pattern 2
[Example]

## Anti-patterns

What to avoid...
```

## Source

Skills are maintained at:

```
github.com/robotdad/skills/
├── playwright/
├── curl/
└── image-vision/
```
