---
id: tool-web
type: tools
title: "Web Tools"
---

# Web Tools

Web tools extend your AI assistant's capabilities beyond local files and code,
enabling real-time information retrieval from the internet. These tools are
essential for research, documentation lookup, API exploration, and staying
current with rapidly evolving technologies.

## Operations

| Tool | Purpose |
|------|---------|
| web_search | Search the internet for information |
| web_fetch | Fetch and parse content from a URL |

Both tools work together: use `web_search` to discover relevant URLs, then
`web_fetch` to retrieve detailed content from specific pages.

---

## Web Search

The `web_search` tool queries the internet and returns relevant results.

### Basic Usage

```
User: Search for Python asyncio best practices
```

The assistant will use `web_search` to find current articles, documentation,
and discussions about the topic.

### When to Use Web Search

- **Documentation lookup**: Find official docs for libraries and frameworks
- **Error resolution**: Search for error messages and solutions
- **Best practices**: Discover current recommendations and patterns
- **News and updates**: Get latest information about technologies
- **Comparative research**: Find comparisons between tools or approaches

### Search Tips

1. **Be specific**: "FastAPI dependency injection tutorial" beats "FastAPI help"
2. **Include context**: Add version numbers or specific use cases
3. **Use quotes**: For exact phrases or error messages
4. **Combine terms**: "React hooks useState useEffect examples"

---

## Fetching URLs

The `web_fetch` tool retrieves content from a specific URL.

### Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| url | string | required | The URL to fetch |
| offset | integer | 0 | Start reading from byte N |
| limit | integer | 204800 | Max bytes to return (200KB) |
| save_to_file | string | null | Save full content to this path |

### Size Limits

By default, `web_fetch` returns up to **200KB** of content. This prevents
overwhelming responses and keeps context manageable.

The response includes metadata to help you understand what was retrieved:

```json
{
  "truncated": true,
  "total_bytes": 450000,
  "content": "..."
}
```

### Using save_to_file

For large pages, save the full content to a file instead of returning it:

```
Fetch https://example.com/large-doc.html and save to ./docs/reference.html
```

This saves the complete content and returns only metadata plus a preview,
keeping your context clean while preserving full access to the data.

### Pagination with offset/limit

For very large content, paginate through it:

```
# First 200KB (default)
web_fetch(url="https://example.com/huge-page")

# Next 200KB
web_fetch(url="https://example.com/huge-page", offset=204800)

# Custom chunk size
web_fetch(url="https://example.com/huge-page", offset=0, limit=50000)
```

---

## Handling Large Pages

Large web pages require special handling to avoid context overflow.

### Strategy 1: Save and Read Selectively

```
1. Fetch URL with save_to_file parameter
2. Read specific sections using read_file with offset/limit
3. Search within saved file using grep
```

This is ideal for documentation pages or API references where you only need
specific sections.

### Strategy 2: Paginate Through Content

```
1. Fetch first chunk (default 200KB)
2. Check if truncated is true
3. Continue fetching with increasing offsets
4. Stop when you find what you need
```

Useful when searching for specific information in a large page.

### Strategy 3: Multiple Targeted Fetches

If a page has anchor links or sections:

```
Fetch https://docs.example.com/api#authentication
Fetch https://docs.example.com/api#endpoints
```

Many documentation sites support fragment navigation.

### Warning: Truncation Breaks Structured Data

When content is truncated, JSON, XML, or other structured formats may break:

```json
{
  "users": [
    {"name": "Alice", "role": "admin"},
    {"name": "Bob", "role": "user"},
    {"name": "Charlie", "ro
[...truncated...]
```

**Workaround**: Use `save_to_file` for any structured data you need to parse.

---

## Handling Redirects

When `web_fetch` encounters a redirect to a different host, it returns a
message indicating the redirect URL. The assistant should then make a new
request with the provided redirect URL.

```
Original: https://short.link/abc123
Redirect: https://actual-destination.com/full-path
```

The assistant handles this automatically in most cases.

---

## Best Practices

### Combine Search and Fetch

```
1. Search: "Pydantic v2 migration guide"
2. Review search results
3. Fetch the most relevant URL for details
```

### Verify Information Currency

Web search results include dates when available. For rapidly evolving
technologies, prefer recent sources:

```
Search: "Python 3.12 new features 2024"
```

### Use Official Sources First

When documentation exists, prefer official sources:

```
Good: Fetch from docs.python.org
Less ideal: Fetch from random blog posts
```

### Cache Large Documentation Locally

For repeated reference to large docs:

```
1. Fetch with save_to_file to ./docs/reference.md
2. Read from local file for subsequent queries
3. Refresh periodically as needed
```

### Respect Rate Limits

Avoid rapid sequential fetches to the same domain. The assistant handles
this automatically, but be aware that some sites may block aggressive access.

---

## Try It Yourself

### Exercise 1: Documentation Lookup

```
Find the official documentation for Python's pathlib module
and explain the difference between Path.resolve() and Path.absolute()
```

### Exercise 2: Error Research

```
Search for: "TypeError: 'NoneType' object is not subscriptable"
and summarize the common causes
```

### Exercise 3: API Documentation

```
Fetch the GitHub REST API documentation for creating issues
and show me the required parameters
```

### Exercise 4: Large Page Handling

```
Fetch a large documentation page and save it locally,
then search for specific sections within the saved file
```

### Exercise 5: Comparative Research

```
Search for comparisons between FastAPI and Flask,
fetch two relevant articles, and summarize the key differences
```

---

## Common Errors

### Connection Errors

| Error | Cause | Solution |
|-------|-------|----------|
| Connection timeout | Server slow or unreachable | Retry or try alternative URL |
| Connection refused | Server not accepting connections | Check URL validity |
| DNS resolution failed | Domain doesn't exist | Verify the domain name |

### HTTP Errors

| Status | Meaning | Action |
|--------|---------|--------|
| 403 Forbidden | Access denied | Site may block automated access |
| 404 Not Found | Page doesn't exist | Verify URL or search for current location |
| 429 Too Many Requests | Rate limited | Wait and retry later |
| 500 Server Error | Server problem | Retry or find alternative source |

### Content Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| Empty response | JavaScript-rendered content | Content may require browser rendering |
| Garbled text | Encoding issues | Usually handled automatically |
| Incomplete data | Truncation | Use save_to_file for full content |

### Blocked Access

Some sites block automated fetching:

- **Paywalled content**: Requires authentication
- **Bot protection**: Sites using Cloudflare or similar
- **Terms of service**: Some sites prohibit scraping

In these cases, try alternative sources or inform the user that direct access
is unavailable.

---

## Summary

Web tools provide powerful internet access capabilities:

- **web_search**: Discover information and find relevant URLs
- **web_fetch**: Retrieve content with size management options

Key techniques:
- Use `save_to_file` for large pages
- Paginate with `offset`/`limit` for huge content
- Combine search and fetch for effective research
- Prefer official documentation sources

These tools transform your assistant into a research partner capable of
finding, retrieving, and synthesizing information from across the web.
