---
id: writing-skills
type: contributing
title: "Writing Skills"
---

# Writing Skills

Skills inject domain expertise into an agent's context on demand. Unlike tools
(which do things) and agents (which delegate things), skills teach things. They're
Markdown files that shape how an agent approaches problems -- your team's best
practices, decision frameworks, and workflows, loadable in a single call.

This guide covers creating, placing, testing, and distributing custom skills.

## The Skill File Format

A skill is a Markdown file with YAML frontmatter following the
[Agent Skills specification](https://agentskills.io). At minimum, it looks like this:

```markdown
---
name: docker-multi-stage
description: >
  Use when building Dockerfiles, optimizing image sizes,
  or debugging container build failures
---

# Docker Multi-Stage Builds

## Core Pattern

Separate build dependencies from runtime for minimal production images.

```dockerfile
FROM node:20 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci && npm run build

FROM node:20-slim
COPY --from=builder /app/dist ./dist
CMD ["node", "dist/index.js"]
```

## Common Mistakes
- Installing dev dependencies in production stage
- Using `latest` tag (breaks reproducibility)
- Running as root in production containers
```

No code, no build step, no registration. Drop a Markdown file in the right
directory and it's discoverable.

## Three-Level Progressive Disclosure

Skills use a token-efficient loading strategy. The agent peels back layers only
as needed:

| Level | What Loads | Token Cost | When |
|-------|-----------|------------|------|
| **1 -- Metadata** | Name + description | ~100 tokens | Always visible via list/search |
| **2 -- Content** | Full markdown body | ~1-5k tokens | Loaded on demand via `load_skill` |
| **3 -- References** | Companion files | 0 until accessed | Read via `skill_directory` path |

This means a session with 30 available skills pays only ~3,000 tokens for
visibility (Level 1 for all). When one skill gets loaded, it adds its content
(Level 2). Companion files (Level 3) load only when the agent explicitly reads
them. Token budget stays under control.

## Frontmatter Fields

### Required Fields

```yaml
name: my-skill              # Unique identifier, used with load_skill
description: >              # WHEN to load this skill (trigger condition)
  Use when writing database migrations
  or debugging schema conflicts
```

### Optional Fields

```yaml
version: 1.0.0              # Semantic version
tags:                        # Discovery aids
  - database
  - migrations
  - postgres
context: fork                # fork = runs as isolated subagent (see below)
context: inline              # inline = loads into current session (default)
disable-model-invocation: true  # User-only, invoked via /command
model_role: reasoning        # Preferred model role for fork execution
```

### Descriptions Are Triggers

The description field determines whether the agent loads your skill. Write it
as a trigger condition, not a summary:

```yaml
# Good: tells the agent WHEN to load
description: Use when building Dockerfiles or debugging build failures

# Bad: summarizes WHAT the skill does
description: Teaches multi-stage Docker builds with layer caching
```

Start with "Use when..." and focus on symptoms and situations the user might
describe. If the description summarizes content, the agent may try to follow
the summary instead of reading the full skill.

## Directory Placement

Skills are discovered from multiple directories, in priority order:

| Location | Scope | Typical Use |
|----------|-------|-------------|
| `.amplifier/skills/` | Workspace | Project-specific patterns |
| `~/.amplifier/skills/` | User (global) | Personal workflows across projects |
| Bundle skill sources | Distribution | Skills shipped with bundles |

Each skill lives in its own directory with a `SKILL.md` file:

```
.amplifier/skills/
└── docker-multi-stage/
    ├── SKILL.md             # Required: main skill content
    ├── examples/            # Optional: companion files
    │   └── basic.dockerfile
    └── templates/           # Optional: reusable templates
        └── multi-stage.dockerfile
```

**First-match-wins**: if the same skill name exists in both workspace and user
directories, the workspace copy takes precedence. This lets you customize shared
skills per-project while keeping personal defaults as fallback.

## Companion Files and skill_directory

When a skill is loaded, the response includes a `skill_directory` path:

```
[Tool: load_skill] skill_name="docker-multi-stage"
→ Skill loaded (1,820 tokens). skill_directory: /path/to/docker-multi-stage/
```

The agent can then read companion files from that directory without loading them
all upfront:

```
[Tool: read_file] file_path="/path/to/docker-multi-stage/examples/basic.dockerfile"
```

This is Level 3 disclosure -- zero tokens until the agent actually needs the
reference material. Use companion files for:

- Code examples and templates
- API reference tables
- Configuration snippets
- Lengthy decision trees

Keep `SKILL.md` itself under ~5,000 tokens. Move heavy reference material into
companion files.

## Enhanced Features

### Fork Execution

Skills with `context: fork` run as isolated subagents rather than loading into
your main context:

```yaml
---
name: architecture-consultant
description: >
  Authoritative consultant for architecture decisions.
  Use when evaluating tradeoffs between approaches.
context: fork
model_role: reasoning
---
```

Fork skills are useful for heavyweight expertise that would overwhelm your main
context window, or for skills that need conversational back-and-forth. The agent
delegates to the fork skill, which works independently and returns results --
similar to delegating to an agent.

The `model_role` field tells Amplifier which model routing to prefer when
executing the fork. Use `reasoning` for analytical skills, `coding` for
implementation-heavy ones.

### User-Invocable Skills (Power Skills)

Skills with `disable-model-invocation: true` aren't loaded by the agent
automatically. Instead, you trigger them via slash commands:

```yaml
---
name: code-review
description: Review changed code for quality and efficiency
disable-model-invocation: true
---
```

Invoke with `/code-review` in a session. Power skills are for workflows you
initiate deliberately -- mass changes, session debugging, structured reviews.
They won't fire on their own.

## Writing Effective Skill Content

Structure your `SKILL.md` for the agent, not a human reader:

```markdown
---
name: api-error-handling
description: >
  Use when implementing API error responses, designing
  error codes, or debugging inconsistent error formats
version: 1.0.0
tags: [api, errors, rest]
---

# API Error Handling Standards

## When to Use
- Implementing new API endpoints with error responses
- Reviewing error handling in existing endpoints
- Debugging inconsistent error formats across services

## Core Pattern
Always return errors in this format:
...

## Decision Framework
| Situation | Status Code | Error Type |
...

## Common Mistakes
- Returning 200 with error body (use proper status codes)
- Leaking stack traces in production responses
- Inconsistent field names across endpoints

## Examples
See companion files for complete examples:
- `examples/validation-errors.json`
- `examples/auth-errors.json`
```

The "When to Use" section reinforces the description. The "Core Pattern" gives
the essential guidance. "Decision Framework" handles conditional logic.
"Common Mistakes" prevents known pitfalls. "Examples" points to companions.

## Testing Your Skill

Place the skill in your workspace and test interactively:

> List available skills.

```
[Tool: load_skill] list=True
→ 24 skills found: ... api-error-handling ...
```

Confirm your skill appears. Then load it:

> Load the api-error-handling skill.

```
[Tool: load_skill] skill_name="api-error-handling"
→ Skill loaded (2,100 tokens). skill_directory: /path/to/api-error-handling/
```

Now test that the agent actually uses it:

> Help me implement error handling for this new POST /users endpoint.

The agent's response should follow patterns from your skill, not generic advice.
If it doesn't, check that your description triggers correctly for the task phrasing
and that the content is specific enough to override default behavior.

To test companion file access:

> Show me the validation error examples from the skill.

```
[Tool: read_file] file_path="/path/to/api-error-handling/examples/validation-errors.json"
```

## Distributing Skills via Bundles

Bundles can register skill directories so their skills appear in discovery:

```yaml
bundle:
  name: my-team-bundle
  version: 1.0.0

skill_sources:
  - path: ./skills
```

All skills in the `./skills` directory (relative to the bundle) become available
when the bundle is loaded. This is how skill collections ship to teams -- package
them in a bundle, publish the bundle, and everyone gets the same expertise.

## Best Practices

**Keep skills focused.** One domain per skill. "React Testing" is good;
"Frontend Development" is too broad. Focused skills are easier to discover and
provide more actionable guidance.

**Write for triggers, not summaries.** Your description answers "when should I
load this?" not "what does this contain?"

**Use companion files for heavy reference.** Keep SKILL.md concise. Move lengthy
API references, examples, and templates into companion files that load only when
needed.

**Workspace overrides user.** Keep personal defaults in `~/.amplifier/skills/`
and project-specific overrides in `.amplifier/skills/`.

**Load on demand, not preemptively.** Each loaded skill consumes context tokens.
The agent should use `info` to check relevance first and load only what the
current task needs.
