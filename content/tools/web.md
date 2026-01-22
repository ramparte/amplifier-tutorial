---
id: tool-web
type: tools
title: "Web Tools"
---

# Web Tools

Web tools enable agents to access and retrieve information from the internet. These tools are essential for gathering real-time data, documentation, and external resources that aren't available in the local context.

## Operations

| Tool | Purpose |
|------|---------|
| `web_search` | Search the internet using a search engine |
| `web_fetch` | Fetch and read content from a specific URL |

## Web Search

The `web_search` tool allows agents to search the internet and retrieve relevant results based on a query.

### Basic Search

```python
web_search(query="Python asyncio tutorial")
```

This returns:
- **Snippets**: Text excerpts from matching pages
- **URLs**: Direct links to the sources
- **Titles**: Page titles for context
- **Metadata**: Relevance scores and timestamps

### Search Parameters

```python
web_search(
    query="FastAPI best practices",
    max_results=10,          # Limit number of results (default: 5)
    search_depth="advanced"  # Search depth: "basic" or "advanced"
)
```

**Search Depth Options:**
- `basic`: Quick search with top results (faster)
- `advanced`: Deeper search with more comprehensive results (slower)

### Use Cases

**Documentation Lookup:**
```python
web_search(query="Django 4.2 ORM query optimization")
```

**API Research:**
```python
web_search(query="OpenAI API rate limits 2024")
```

**Error Troubleshooting:**
```python
web_search(query="Python ModuleNotFoundError importlib.metadata")
```

**Version-Specific Information:**
```python
web_search(query="Node.js 20 breaking changes")
```

### Search Query Tips

1. **Be Specific**: Include version numbers, library names, or exact error messages
2. **Use Quotes**: For exact phrase matching: `"cannot import name"`
3. **Include Year**: For time-sensitive information: `"React hooks 2024"`
4. **Filter by Site**: Use `site:` operator: `"site:stackoverflow.com python async"`

## Fetching URLs

The `web_fetch` tool retrieves the full content of a specific web page.

### Basic Fetch

```python
web_fetch(url="https://docs.python.org/3/library/asyncio.html")
```

Returns the page content in a readable text format, stripped of HTML tags and formatting.

### Size Limits

Web pages can be very large. The tool has built-in protections:

- **Default Limit**: ~200KB of text content
- **Truncation**: Large pages are automatically truncated
- **Warning**: You'll be notified when content is truncated

**Handling Truncated Content:**
```python
# For large pages, save to file instead
web_fetch(
    url="https://example.com/long-article",
    save_to_file="/tmp/article.txt"
)
```

### Save to File

For large documents, API responses, or content you need to process:

```python
web_fetch(
    url="https://api.github.com/repos/python/cpython",
    save_to_file="/tmp/github_response.json"
)
```

**Benefits:**
- No content truncation
- Can process file in chunks
- Preserves exact formatting
- Efficient memory usage

### Fetching Different Content Types

**HTML Pages:**
```python
web_fetch(url="https://developer.mozilla.org/en-US/docs/Web/JavaScript")
# Returns cleaned text content
```

**API Endpoints:**
```python
web_fetch(
    url="https://api.example.com/data",
    save_to_file="/tmp/api_response.json"
)
```

**Raw Files:**
```python
web_fetch(
    url="https://raw.githubusercontent.com/user/repo/main/README.md",
    save_to_file="/tmp/readme.md"
)
```

**Documentation:**
```python
web_fetch(url="https://fastapi.tiangolo.com/tutorial/")
```

## Handling Large Pages

When dealing with large web pages, use a strategic approach:

### Strategy 1: Search First, Fetch Later

```python
# 1. Search for the topic
results = web_search(query="Python decorator patterns")

# 2. Review snippets to find most relevant
# 3. Fetch only the most promising URL
web_fetch(url=results[0]['url'], save_to_file="/tmp/decorators.txt")

# 4. Read specific sections of the saved file
read_file("/tmp/decorators.txt", offset=1, limit=100)
```

### Strategy 2: Fetch to File

```python
# Fetch large content to file
web_fetch(
    url="https://docs.djangoproject.com/en/5.0/",
    save_to_file="/tmp/django_docs.html"
)

# Read relevant sections
read_file("/tmp/django_docs.html", offset=100, limit=50)
```

### Strategy 3: Multiple Targeted Fetches

Instead of fetching one large page:
```python
# Fetch specific sub-pages
web_fetch(url="https://example.com/docs/intro")
web_fetch(url="https://example.com/docs/api-reference")
web_fetch(url="https://example.com/docs/examples")
```

## Best Practices

### 1. Search Before Fetching

Don't fetch URLs blindly. Use search to find the right content first:

```python
# Good: Search first to find relevant content
results = web_search(query="FastAPI middleware authentication")
web_fetch(url=results[0]['url'])

# Avoid: Fetching without knowing if it's relevant
web_fetch(url="https://example.com/docs/")  # Might be huge and irrelevant
```

### 2. Use Appropriate Tools

**Use `web_search` when:**
- You need to find information but don't have a URL
- You want multiple perspectives or sources
- You're researching current events or trends
- You need to verify information across sources

**Use `web_fetch` when:**
- You have a specific URL to retrieve
- You need the complete content of a page
- You're accessing API endpoints
- You need to download files or raw content

### 3. Handle Failures Gracefully

```python
# Web operations can fail due to network issues, timeouts, etc.
# Always have a fallback strategy

# Try fetching documentation
try:
    web_fetch(url="https://docs.example.com/api")
except:
    # Fall back to search if fetch fails
    web_search(query="example.com API documentation")
```

### 4. Be Respectful

- Don't fetch the same URL repeatedly in a short time
- Use search when possible to minimize direct fetches
- Save large content to files to avoid re-fetching

### 5. Verify Information

Web content can be outdated or incorrect:
- Check publication dates in search results
- Cross-reference information from multiple sources
- Prefer official documentation over third-party sources

## Try It Yourself

### Exercise 1: Research a Library

Search for information about a Python library and fetch its documentation:

```python
# 1. Search for the library
web_search(query="httpx Python async client tutorial")

# 2. Fetch the official documentation
web_fetch(url="https://www.python-httpx.org/")

# 3. Search for specific examples
web_search(query="httpx async examples GitHub")
```

### Exercise 2: API Investigation

Research an API and fetch example responses:

```python
# 1. Search for API documentation
web_search(query="GitHub REST API repositories endpoint")

# 2. Fetch the API documentation
web_fetch(url="https://docs.github.com/en/rest/repos/repos")

# 3. Fetch example data and save it
web_fetch(
    url="https://api.github.com/repos/python/cpython",
    save_to_file="/tmp/github_api_example.json"
)
```

### Exercise 3: Troubleshooting

Find solutions to a specific error:

```python
# 1. Search for the error message
web_search(query="Python 'RuntimeError: Event loop is closed' asyncio")

# 2. Fetch the most relevant Stack Overflow answer
web_fetch(url="<most_relevant_stackoverflow_url>")

# 3. Search for related issues
web_search(query="asyncio event loop best practices")
```

## Common Errors

### "Failed to fetch URL"

**Cause**: Network issues, invalid URL, or server unavailable

**Solution**:
- Verify the URL is correct and accessible
- Check if the website is online
- Try searching for the content instead
- Wait and retry if it's a temporary issue

### "Content truncated due to size"

**Cause**: Web page exceeds size limits

**Solution**:
```python
# Save to file instead
web_fetch(url="<large_url>", save_to_file="/tmp/content.txt")
```

### "Rate limit exceeded"

**Cause**: Too many requests in a short time

**Solution**:
- Space out your requests
- Use search results without fetching every URL
- Cache previously fetched content

### "Search returned no results"

**Cause**: Query too specific or obscure topic

**Solution**:
- Broaden your search terms
- Remove version numbers or very specific details
- Try alternative phrasings
- Check spelling

## Summary

Web tools are powerful for:
- **Research**: Finding current information and best practices
- **Documentation**: Accessing official docs and API references
- **Troubleshooting**: Finding solutions to errors and issues
- **Data Collection**: Retrieving content from web sources

**Key Points:**
- Use `web_search` to find information, `web_fetch` to retrieve specific content
- Save large content to files to avoid truncation
- Search first, then fetch to be efficient
- Handle failures gracefully and verify information
- Be respectful with request frequency

Web tools bridge the gap between local knowledge and the vast resources available online, enabling agents to access the most current and comprehensive information available.
