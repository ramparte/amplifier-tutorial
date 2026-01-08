---
id: skills
type: concepts
title: "Skills"
---

# Skills

Skills are reusable knowledge packages that give Amplifier domain expertise. They follow the Anthropic Skills format - structured markdown that teaches the AI how to approach specific domains.

## What is a Skill?

A skill provides:

- **Workflows** - Decision trees for complex tasks
- **Patterns** - Best practices and examples
- **Troubleshooting** - Common errors and solutions
- **Setup guides** - Installation and configuration

Unlike tools (which do things), skills teach the AI *how* to think about problems.

## Loading Skills

```bash
# In an Amplifier session
amplifier> Load the playwright skill

# Or explicitly
amplifier> /skill load playwright
```

Once loaded, the AI has domain expertise:

```
amplifier> I need to test that the login form works

# With playwright skill loaded, the AI knows:
# - Use headless mode by default
# - Prefer role-based selectors
# - Handle auth state properly
# - Capture diagnostics on failure
```

## Available Skills

### Playwright (Browser Automation)

```
robotdad/skills/playwright/
├── SKILL.md           # Core workflow and decision tree
├── patterns.md        # Advanced patterns
├── setup.md           # Installation
└── troubleshooting.md # Common issues
```

**What it teaches:**
- Headless-first approach (never steal focus)
- Role-based selector strategy
- Explicit wait patterns
- Diagnostic capture (screenshots, traces)
- Authentication handling
- Multi-page workflows

### Curl (HTTP Client)

```
robotdad/skills/curl/
├── SKILL.md
├── patterns.md
└── troubleshooting.md
```

**What it teaches:**
- REST API testing
- GraphQL queries
- Request/response validation
- Authentication patterns
- Error handling

### Image Vision (Visual Analysis)

```
robotdad/skills/image-vision/
├── SKILL.md
├── patterns.md
└── setup.md
```

**What it teaches:**
- Image content understanding
- OCR text extraction
- Visual comparison
- Multi-image analysis
- Provider selection (Anthropic, OpenAI, etc.)

## Skill Structure

A skill is a directory with markdown files:

```
my-skill/
├── SKILL.md           # Required: Main skill definition
├── patterns.md        # Optional: Advanced patterns
├── setup.md           # Optional: Setup instructions
└── troubleshooting.md # Optional: Common issues
```

### SKILL.md Format

```markdown
# Skill Name

Brief description of what this skill enables.

## When to Use

- Scenario 1
- Scenario 2
- Scenario 3

## Workflow

```
START
├── Check prerequisites
│   ├── Yes → Proceed to step 2
│   └── No → Run setup
├── Analyze the task
│   ├── Type A → Use pattern A
│   └── Type B → Use pattern B
└── Execute
    └── Capture results
```

## Core Patterns

### Pattern 1: Basic Usage

[Example code and explanation]

### Pattern 2: Advanced Usage

[Example code and explanation]

## Anti-patterns

Things to avoid:
- Don't do X because Y
- Never assume Z

## Quick Reference

| Command | Purpose |
|---------|---------|
| `cmd1`  | Does X  |
| `cmd2`  | Does Y  |
```

## Progressive Disclosure

Skills use a token-efficient pattern:

| Level | Content | Token Cost |
|-------|---------|------------|
| **1** | Metadata (name, description) | ~100 tokens |
| **2** | Full SKILL.md | ~1,000 tokens |
| **3** | Reference files (patterns, setup) | Loaded on demand |

The AI loads only what's needed:

```
# Simple task → Level 1-2 only
"Take a screenshot of google.com"

# Complex task → Level 1-3
"Set up playwright with auth state and parallel browser contexts"
```

## Creating Custom Skills

### Step 1: Create Skill Directory

```bash
mkdir -p my-skills/docker-expert
```

### Step 2: Write SKILL.md

```markdown
# Docker Expert

Expert knowledge for Docker containerization and orchestration.

## When to Use

- Building Dockerfiles
- Debugging container issues
- Optimizing image sizes
- Docker Compose workflows

## Workflow

```
START
├── Is this a build issue?
│   ├── Yes → Check Dockerfile patterns
│   └── No → Continue
├── Is this a runtime issue?
│   ├── Yes → Check logs and networking
│   └── No → Continue
└── Is this a compose issue?
    └── Check service dependencies
```

## Core Patterns

### Multi-stage Builds

Always use multi-stage builds to minimize image size:

```dockerfile
# Build stage
FROM node:20 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Production stage
FROM node:20-slim
WORKDIR /app
COPY --from=builder /app/dist ./dist
CMD ["node", "dist/index.js"]
```

### Layer Caching

Order commands from least to most frequently changing:

```dockerfile
# Least changing
COPY package*.json ./
RUN npm ci

# Most changing
COPY src/ ./src/
```

## Anti-patterns

- Don't run as root in production
- Don't use `latest` tag
- Don't install dev dependencies in production image
```

### Step 3: Register the Skill

In your bundle:

```yaml
skills:
  - path: ./my-skills/docker-expert
```

### Step 4: Use It

```bash
amplifier> Load the docker-expert skill
amplifier> Review this Dockerfile for best practices
```

## Skills vs Context Files

| Aspect | Skill | Context File |
|--------|-------|--------------|
| **Format** | Structured markdown | Free-form |
| **Loading** | On-demand, progressive | All at once |
| **Purpose** | Teach approach/workflow | Provide information |
| **Scope** | Domain expertise | Specific facts |

Use skills for *how to think*, context files for *what to know*.

## Try It Yourself

### Exercise 1: Load a Skill

```bash
amplifier

> Load the playwright skill
> Now help me write a test that logs into a website
```

### Exercise 2: Compare With/Without

```bash
# Without skill
amplifier run "Write a playwright test for form validation"

# With skill
amplifier
> Load the playwright skill
> Write a playwright test for form validation
```

Notice the difference in approach and best practices.

### Exercise 3: Explore Skill Contents

```bash
# The skill files are just markdown - you can read them
cat ~/.amplifier/skills/playwright/SKILL.md
```

## Key Takeaways

1. **Skills teach approach** - They're knowledge, not tools
2. **Progressive loading** - Only what's needed is loaded
3. **Structured format** - Workflows, patterns, troubleshooting
4. **Reusable** - Share skills across projects and teams

## Next

Learn about lifecycle observation:

→ [Hooks](hooks.md)
