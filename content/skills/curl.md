---
id: skill-curl
type: skills
title: "Curl Skill"
---

# Curl Skill

HTTP client for API testing, validation, and debugging.

## Overview

The curl skill teaches Amplifier how to:

- Test REST APIs
- Validate responses
- Debug HTTP requests
- Send webhooks
- Handle authentication

## Loading the Skill

```
> Load the curl skill
```

Once loaded, Amplifier knows HTTP best practices.

## When to Use

| Task | Use Curl |
|------|----------|
| Test API endpoint | ✅ |
| Send webhook | ✅ |
| Debug HTTP issue | ✅ |
| Validate JSON response | ✅ |
| Navigate website | ❌ Use playwright |
| Fill web form | ❌ Use playwright |

## Core Patterns

### Basic GET

```bash
curl https://api.example.com/users
```

### POST with JSON

```bash
curl -X POST https://api.example.com/users \
  -H "Content-Type: application/json" \
  -d '{"name": "Alice", "email": "alice@example.com"}'
```

### With Authentication

```bash
# Bearer token
curl https://api.example.com/me \
  -H "Authorization: Bearer $TOKEN"

# Basic auth
curl -u username:password https://api.example.com/private
```

### Show Headers

```bash
curl -I https://api.example.com/health  # Headers only
curl -i https://api.example.com/users   # Headers + body
```

### Verbose Mode

```bash
curl -v https://api.example.com/debug
```

## Response Validation

### Check Status Code

```bash
# Get just the status code
curl -o /dev/null -s -w "%{http_code}" https://api.example.com/health
```

### Validate JSON

```bash
# Pretty print JSON
curl https://api.example.com/users | jq .

# Extract specific field
curl https://api.example.com/users/1 | jq '.name'

# Check array length
curl https://api.example.com/users | jq 'length'
```

### Check Response Time

```bash
curl -o /dev/null -s -w "Time: %{time_total}s\n" https://api.example.com/
```

## Common Methods

### GET

```bash
curl https://api.example.com/resource
curl "https://api.example.com/search?q=term"
```

### POST

```bash
curl -X POST https://api.example.com/resource \
  -H "Content-Type: application/json" \
  -d '{"key": "value"}'
```

### PUT

```bash
curl -X PUT https://api.example.com/resource/1 \
  -H "Content-Type: application/json" \
  -d '{"name": "Updated"}'
```

### DELETE

```bash
curl -X DELETE https://api.example.com/resource/1
```

### PATCH

```bash
curl -X PATCH https://api.example.com/resource/1 \
  -H "Content-Type: application/json" \
  -d '{"status": "active"}'
```

## Headers

### Common Headers

```bash
# Content-Type
-H "Content-Type: application/json"
-H "Content-Type: application/x-www-form-urlencoded"

# Accept
-H "Accept: application/json"

# Authorization
-H "Authorization: Bearer TOKEN"
-H "Authorization: Basic BASE64"

# Custom
-H "X-API-Key: your-key"
```

### Multiple Headers

```bash
curl https://api.example.com/ \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Accept: application/json"
```

## File Operations

### Upload File

```bash
curl -X POST https://api.example.com/upload \
  -F "file=@document.pdf"
```

### Download File

```bash
curl -o output.pdf https://example.com/document.pdf
```

## Error Handling

### Follow Redirects

```bash
curl -L https://example.com/redirecting-url
```

### Retry on Failure

```bash
curl --retry 3 https://api.example.com/flaky
```

### Timeout

```bash
curl --connect-timeout 5 --max-time 10 https://api.example.com/slow
```

## Debugging

### Verbose Output

```bash
curl -v https://api.example.com/debug
```

Shows:
- DNS resolution
- TCP connection
- TLS handshake
- Request headers
- Response headers

### Trace

```bash
curl --trace - https://api.example.com/debug
```

Full byte-level trace.

## Try It Yourself

### Exercise 1: Test an API

```
> Load curl skill
> Test the GitHub API: GET https://api.github.com/users/octocat
```

### Exercise 2: POST Request

```
> Send a POST to httpbin.org/post with JSON body {"test": "data"}
```

### Exercise 3: Validate Response

```
> Check if https://api.github.com is returning 200 OK
```

## Common Errors

### "Connection refused"

- Server not running
- Wrong port
- Firewall blocking

### "SSL certificate problem"

```bash
# Skip verification (only for testing!)
curl -k https://self-signed.example.com
```

### "401 Unauthorized"

- Check authentication header
- Verify token is valid
- Check token format

## Source

```
robotdad/skills/curl/
├── SKILL.md           # Core patterns
├── patterns.md        # Advanced usage
└── troubleshooting.md # Common issues
```
