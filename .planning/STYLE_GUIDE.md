# Tutorial Style Guide

This document defines how tutorial content should be written for consistency and pedagogical effectiveness.

## Core Philosophy

**This is a NARRATIVE TUTORIAL, not reference documentation.**

The goal is to teach users HOW TO USE Amplifier through conversation, not how to configure modules as a developer.

## Voice and Tone

- **Conversational** - Write as if explaining to a colleague
- **User-facing** - Focus on what users TYPE, not what developers configure
- **Progressive** - Start simple, add complexity gradually
- **Hands-on** - Every concept has a try-it example

## Content Structure for Tool Pages

### Required Sections

```markdown
---
id: tool-[name]
type: tools
title: "[Tool Name]"
---

# [Tool Name]

[One-line description of what it does for the USER]

## Overview

[Brief table or list of operations with "When to Use" guidance]

## [Operation 1]

### Basic Usage

```
> [Natural language prompt the user would type]
```

Behind the scenes:
```
[Tool: operation_name]
parameter: value
```

### [Variation 1]

[Show more complex usage with user prompt + tool call]

## [Operation 2]
...

## Best Practices

[Bullet points of how to use effectively]

## Common Patterns

[Code snippets for frequent use cases]

## Try It Yourself

### Exercise 1: [Name]
```
> [User prompt to try]
```

### Exercise 2: [Name]
...

## Errors and Troubleshooting

### "[Error message]"
[Explanation and solution]
```

## Example Patterns

### Showing User Interaction

ALWAYS show what the USER types, then what happens:

```markdown
### Basic Usage

```
> Show me the contents of config.yaml
```

Behind the scenes:
```
[Tool: read_file]
file_path: config.yaml
```
```

### NOT This (Developer/Config Focus)

```markdown
### Configuration

```toml
[[tools]]
module = "tool-filesystem"
config = { allowed_paths = ["."] }
```
```

### Progressive Examples

Start with the simplest case, then add complexity:

1. **Basic** - `> Show me config.yaml`
2. **With options** - `> Show me lines 100-150 of app.py`  
3. **Edge case** - `> What files are in src/`

### Exercises

Make them achievable and verify-able:

```markdown
## Try It Yourself

### Exercise 1: Create and Read

```
> Create a file called hello.py that prints "Hello, World!"
> Now show me what's in hello.py
```

### Exercise 2: Edit in Place

```
> Change hello.py to print "Hello, Amplifier!" instead
```
```

### Troubleshooting

Show actual error messages users will see:

```markdown
### "File not found"

The file doesn't exist at that path. Check:
- Spelling of filename
- Working directory
- Relative vs absolute path
```

## What NOT to Include

- TOML/YAML configuration examples (that's developer docs)
- Module installation commands (`uv add ...`)
- Provider setup instructions (belongs in quickstart)
- Internal architecture details (belongs in concepts)

## Frontmatter Schema

```yaml
---
id: tool-[name]           # kebab-case identifier
type: tools               # tools | bundles | concepts | quickstart | skills | advanced
title: "[Display Name]"   # Human-readable title
---
```

Note: The full schema with `source_repo`, `last_synced`, `dependencies`, etc. is optional for generation. The recipes will add these fields.

## Length Guidelines

- **Tool pages**: 150-300 lines (like filesystem.md at 269 lines)
- **Concept pages**: 100-200 lines
- **Quickstart pages**: 150-250 lines

## Code Block Languages

- User prompts: no language (plain text with `>` prefix)
- Tool calls: no language (shows `[Tool: name]` format)
- Shell commands: `bash`
- Python code: `python`
- JavaScript: `javascript`
- YAML config (only when showing output): `yaml`
