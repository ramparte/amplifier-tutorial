---
id: debugging
type: dev-setup
title: "Debugging"
---

# Debugging

Troubleshoot issues with Amplifier and your AI-assisted workflows.

## Overview

Debugging approaches:

| Issue Type | Where to Look |
|------------|---------------|
| Session errors | `events.jsonl` |
| Tool failures | Tool output, logs |
| Provider issues | API errors, model limits |
| Recipe problems | Step results, context |
| Bundle issues | Bundle validation |

## Session Analysis

### Find Your Session

```bash
# List recent sessions
ls -lt ~/.amplifier/sessions/ | head -10

# Or in Amplifier
amp session list
```

### Read Session Events

Sessions are logged to `events.jsonl`:

```bash
cd ~/.amplifier/sessions/[session-id]

# View all events
cat events.jsonl | jq .

# Filter by event type
cat events.jsonl | jq 'select(.event_type == "tool:call")'

# See errors
cat events.jsonl | jq 'select(.event_type | contains("error"))'
```

### Common Event Types

| Event | Meaning |
|-------|---------|
| `session:start` | Session began |
| `turn:start` | User message received |
| `provider:request` | API call to LLM |
| `provider:response` | LLM response |
| `tool:call` | Tool invoked |
| `tool:result` | Tool returned |
| `turn:end` | Response complete |
| `session:end` | Session ended |

### Session Analyst Agent

For complex issues, use the specialist:

```
> Analyze why session [id] failed
```

The `session-analyst` agent safely handles large event logs.

## Tool Debugging

### See Tool Calls

```bash
# In session, see what tools were called
cat events.jsonl | jq -r 'select(.event_type == "tool:call") | .tool_name'
```

### Tool Errors

```bash
# Find tool errors
cat events.jsonl | jq 'select(.event_type == "tool:result" and .error != null)'
```

### Common Tool Issues

**bash timeout:**
```bash
# Command took too long
# Solution: Use explicit timeout
timeout 120 long_command
```

**File not found:**
```bash
# Check path
ls -la [path]

# Check working directory
pwd
```

**Permission denied:**
```bash
# Check permissions
ls -la [file]
chmod +x [script]
```

## Provider Debugging

### API Errors

```bash
# Find provider errors
cat events.jsonl | jq 'select(.event_type == "provider:response" and .error != null)'
```

### Common Provider Issues

**Rate limiting:**
```
Error: 429 Too Many Requests
```
- Wait and retry
- Check your API plan limits
- Consider using a different model

**Token limits:**
```
Error: Context length exceeded
```
- Conversation too long
- Start new session
- Use smaller context

**Invalid API key:**
```
Error: 401 Unauthorized
```
- Check `ANTHROPIC_API_KEY` is set
- Verify key is valid
- Check for typos

### Test Provider Connection

```bash
# Anthropic
curl https://api.anthropic.com/v1/messages \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "content-type: application/json" \
  -H "anthropic-version: 2023-06-01" \
  -d '{"model": "claude-sonnet-4-20250514", "max_tokens": 10, "messages": [{"role": "user", "content": "Hi"}]}'
```

## Recipe Debugging

### Validate Recipe

```bash
amp recipes validate my-recipe.yaml
```

### Check Recipe Session

```bash
# List recipe sessions
amp recipes list

# View recipe session events
cat ~/.amplifier/sessions/recipe_[id]/events.jsonl | jq .
```

### Step Results

```bash
# See what each step returned
cat events.jsonl | jq 'select(.event_type == "step:complete")'
```

### Common Recipe Issues

**Context variable missing:**
```
Error: Required context variable 'file_path' not provided
```
- Add `--context '{"file_path": "..."}'`

**Step failed:**
- Check the step's agent
- Review the instruction
- Look at step result in events

## Bundle Debugging

### Validate Bundle

```bash
amp bundle validate ./my-bundle
```

### Check What's Loaded

```
> What bundle am I using?
> What tools do you have?
> What agents are available?
```

### Bundle Load Errors

Check for:
- Missing `bundle.yaml`
- Invalid YAML syntax
- Missing included bundles
- Invalid module references

## Verbose Mode

Get more output:

```bash
# Set debug logging
export AMPLIFIER_LOG_LEVEL=debug

# Run Amplifier
amp
```

## Log Locations

| Log | Location |
|-----|----------|
| Session events | `~/.amplifier/sessions/[id]/events.jsonl` |
| Application logs | `~/.amplifier/logs/` |
| Recipe sessions | `~/.amplifier/sessions/recipe_[id]/` |

## Debugging Workflow

### Step 1: Reproduce

```bash
# Note the session ID
amp session list | head -1
```

### Step 2: Find Events

```bash
cd ~/.amplifier/sessions/[id]
cat events.jsonl | jq '.event_type' | sort | uniq -c
```

### Step 3: Locate Error

```bash
# Find errors
cat events.jsonl | jq 'select(.error != null)'

# Find last event before error
cat events.jsonl | tail -20 | jq .
```

### Step 4: Understand Context

```bash
# See what led to error
cat events.jsonl | jq 'select(.event_type == "turn:start" or .event_type == "tool:call")' | tail -10
```

### Step 5: Fix and Retry

Based on findings:
- Fix configuration
- Adjust prompts
- Retry operation

## Try It Yourself

### Exercise 1: Explore Session

```bash
# Find latest session
SESSION=$(ls -t ~/.amplifier/sessions/ | head -1)

# Count event types
cat ~/.amplifier/sessions/$SESSION/events.jsonl | jq '.event_type' | sort | uniq -c
```

### Exercise 2: Find Tool Calls

```bash
# What tools were used?
cat ~/.amplifier/sessions/$SESSION/events.jsonl | \
  jq -r 'select(.event_type == "tool:call") | .tool_name' | \
  sort | uniq -c | sort -rn
```

### Exercise 3: Debug an Error

Intentionally cause an error:
```
> Read the file /nonexistent/path/file.txt
```

Then find it in logs:
```bash
cat events.jsonl | jq 'select(.error != null)'
```

## Getting Help

If stuck:

1. **Search the error** - Often others have hit it
2. **Check GitHub Issues** - Known problems and solutions
3. **Ask Amplifier** - "Why did this fail?"
4. **Session analyst** - "Analyze session [id]"
