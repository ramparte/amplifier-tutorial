---
id: tool-web
type: tools
title: "Web Tools"
---

# Web Tools

Search the internet and fetch content from URLs.

## Overview

| Tool | Purpose | Use When |
|------|---------|----------|
| `web_search` | Search the internet | Find docs, examples, news |
| `web_fetch` | Get URL content | Read specific pages |

## web_search

Search the internet for information.

### Basic Search

```
> What's new in Python 3.12?
```

```
[Tool: web_search]
query: "Python 3.12 new features"
```

### Targeted Searches

```
# Documentation
> Find the FastAPI documentation for dependencies
[web_search] query: "FastAPI dependency injection documentation"

# Error solutions
> How to fix "ModuleNotFoundError: No module named 'xyz'"
[web_search] query: "Python ModuleNotFoundError xyz solution"

# Latest versions
> What's the latest version of React?
[web_search] query: "React latest version 2026"
```

### Search Tips

**Be specific:**
```
# Vague
query: "python error"

# Specific
query: "Python TypeError: 'NoneType' object is not subscriptable fix"
```

**Include context:**
```
query: "FastAPI SQLAlchemy async session best practices"
```

**Add year for current info:**
```
query: "best Python web framework 2026"
```

## web_fetch

Fetch content from a specific URL.

### Basic Fetch

```
> Get the README from that GitHub repo
```

```
[Tool: web_fetch]
url: "https://github.com/microsoft/amplifier/blob/main/README.md"
```

### Fetching Documentation

```
> Read the FastAPI quickstart guide
```

```
[Tool: web_fetch]
url: "https://fastapi.tiangolo.com/tutorial/"
```

### Large Pages

For large content, save to file:

```
[Tool: web_fetch]
url: "https://example.com/large-doc"
save_to_file: "/tmp/doc.html"
```

### Pagination

For very large pages, paginate:

```
# First 100KB
[web_fetch] url: "..." limit: 102400

# Next 100KB
[web_fetch] url: "..." offset: 102400 limit: 102400
```

## Common Workflows

### Research → Implement

```
You: How do I implement OAuth2 in FastAPI?

Amplifier:
1. [web_search] "FastAPI OAuth2 implementation guide"
2. [web_fetch] (relevant documentation URL)
3. Here's how to implement it: [explanation with code]
```

### Debug → Search → Fix

```
You: I'm getting this error: [paste error]

Amplifier:
1. [web_search] "error message solution"
2. Based on what I found, here's the fix...
```

### Check Latest

```
You: Is there a newer version of pandas?

Amplifier:
1. [web_search] "pandas latest version pypi"
2. [web_fetch] PyPI page
3. The latest version is X.Y.Z
```

## What Gets Returned

### Search Results

- Title and URL of matching pages
- Brief snippets
- Typically 5-10 results

### Fetched Content

- Cleaned text content (HTML stripped)
- Limited to ~200KB by default
- Metadata (truncated flag, total bytes)

## Rate Limits and Caching

- Searches and fetches may be rate-limited
- Results may be cached briefly
- Very large pages may be truncated

## Try It Yourself

### Exercise 1: Research a Topic

```
> What are the best practices for Python type hints in 2026?
```

### Exercise 2: Fetch Documentation

```
> Get the README from https://github.com/python/cpython
```

### Exercise 3: Debug an Error

```
> Search for how to fix "CORS error in fetch request"
```

## When NOT to Use Web Tools

### Use grep Instead

```
# Looking for code in YOUR project
# Don't search the web - search locally
[grep] pattern: "authenticate"
```

### Use read_file Instead

```
# Reading YOUR files
# Don't fetch - read directly
[read_file] file_path: README.md
```

### Use Skills Instead

```
# For domain expertise (playwright, curl, etc.)
# Don't search - load the skill
> Load the playwright skill
```

## Errors and Troubleshooting

### "URL not accessible"

- Check URL is valid
- Site may be blocking automated access
- Try a different URL

### "Content truncated"

- Large page - use `save_to_file`
- Or use `offset`/`limit` to paginate

### "No results found"

- Try different search terms
- Be more specific or more general
- Check spelling

### Redirects

If fetch returns a redirect message:

```
[web_fetch] url: "redirect-target-url"
```

Follow the redirect URL provided.
