---
id: tool-web
type: tool
title: "Web Tools"
---

# Web Tools

Not everything you need lives on disk. Sometimes you need to check what version of a library just shipped, look up an API you've never used, or pull down documentation that only exists on the web. Amplifier gives you two tools for this: `web_search` to find things, and `web_fetch` to retrieve them. Together, they're how agents bridge the gap between local context and the rest of the internet.

## What Are the Web Tools?

| Tool | What It Does | Think of It As |
|------|-------------|----------------|
| `web_search` | Search the web for information | Typing a query into a search engine |
| `web_fetch` | Fetch content from a specific URL | Opening a link in your browser |

They're intentionally simple. Search returns results with titles, snippets, and URLs. Fetch retrieves the content at a URL and hands it back as text. Most web work is just those two steps: find the right page, then read it.

## Core Capabilities

### web_search — Finding Things

When you need information but don't have a URL, `web_search` queries the web and returns results:

> What's the latest version of FastAPI?

```
[Tool: web_search] "FastAPI latest version release"
Results:
  1. FastAPI 0.115.0 Release Notes - tiangolo.com
     FastAPI 0.115.0 has been released with support for...
  2. FastAPI - PyPI
     Latest version: 0.115.0 (released Oct 2025)
```

The tool takes one parameter — `query` — and that's it. The quality of your results depends entirely on how you phrase the search, just like a real search engine.

### web_fetch — Retrieving Content

Once you have a URL, `web_fetch` pulls down the content:

> Fetch the FastAPI dependency injection docs

```
[Tool: web_fetch] https://fastapi.tiangolo.com/tutorial/dependencies/
Content (42KB):
  Dependencies - First Steps
  FastAPI has a very powerful but intuitive Dependency Injection system...
```

The response includes a few key fields you should know about:

- **Content** — the page text, stripped of HTML markup
- **truncated** — a boolean telling you whether the content was cut short
- **total_bytes** — the original page size, so you can gauge what you're missing

**The 200KB default limit.** Pages larger than 200KB are automatically truncated to keep your conversation context manageable. The response tells you when this happens and how large the original was, so you can decide what to do next.

**Saving large content to a file.** When you need the full content of a big page, use `save_to_file` to write it to disk instead of returning it inline:

> Fetch the full Python asyncio docs and save them

```
[Tool: web_fetch] https://docs.python.org/3/library/asyncio.html
  save_to_file: /tmp/asyncio-docs.html
Saved 380KB to /tmp/asyncio-docs.html (preview: first 500 bytes shown)
```

Now you can read it in controlled chunks with `read_file`, searching for the section you actually care about.

**Pagination with offset and limit.** For large content you can also paginate directly, fetching byte ranges instead of the whole page:

> Get the next chunk of that page

```
[Tool: web_fetch] https://example.com/large-api-reference
  offset: 204800
  limit: 204800
```

This picks up where the previous fetch left off — useful when you're working through a long reference page section by section.

## Practical Examples

### Looking Up Documentation

The most common pattern is search-then-fetch. You search for the topic, scan the snippets to find the most relevant result, then fetch that page:

> How do I use Python's dataclasses with default_factory?

```
[Tool: web_search] "Python dataclasses default_factory example"
  1. dataclasses — Data Classes - Python docs
  2. Understanding default_factory in Python dataclasses - Real Python

[Tool: web_fetch] https://docs.python.org/3/library/dataclasses.html
```

Amplifier reads the search results, picks the official docs, and fetches them — all in one turn.

### Fetching Raw GitHub Content

GitHub's `raw.githubusercontent.com` URLs are ideal for fetching source files, READMEs, or config examples directly:

> Grab the pyproject.toml from the httpx repo

```
[Tool: web_fetch] https://raw.githubusercontent.com/encode/httpx/master/pyproject.toml
  save_to_file: /tmp/httpx-pyproject.toml
Saved 2.4KB to /tmp/httpx-pyproject.toml
```

This works cleanly because raw GitHub URLs return plain text — no HTML to strip, no JavaScript to execute. It's the fastest way to inspect a file from a public repo without cloning it.

### Researching a Library Before Adopting It

When you're evaluating whether to use a dependency, web tools let you gather context quickly:

> Research the `pydantic-settings` library — is it well-maintained? What does it do?

```
[Tool: web_search] "pydantic-settings library Python"
[Tool: web_fetch] https://pypi.org/project/pydantic-settings/
[Tool: web_fetch] https://docs.pydantic.dev/latest/concepts/pydantic_settings/

Based on the docs and PyPI page:
- Last release: 2 weeks ago, actively maintained
- Provides settings management using Pydantic models
- Supports .env files, environment variables, and secrets...
```

### Fetching API Documentation

API references are often large. Save them to a file so you can search through them:

> Get the Stripe API docs for the charges endpoint

```
[Tool: web_search] "Stripe API create charge documentation"
[Tool: web_fetch] https://docs.stripe.com/api/charges/create
  save_to_file: /tmp/stripe-charges.txt
Saved 95KB to /tmp/stripe-charges.txt

[Tool: read_file] /tmp/stripe-charges.txt (lines 1-80)
```

The fetch-save-read pattern keeps large documentation out of your conversation context while still making it fully accessible.

## Web Tools vs. Browser Agents

The web tools handle static content — pages where the information is in the HTML that comes back from the server. But the modern web is full of JavaScript-rendered applications where the raw HTML is mostly empty scaffolding. Here's how to decide:

| Scenario | Best Approach | Why |
|----------|--------------|-----|
| Static HTML pages, docs, blog posts | **web_fetch** | Content is in the HTML response |
| Raw files, API endpoints, JSON | **web_fetch** | Direct content, no rendering needed |
| JavaScript SPAs, dynamic dashboards | **Browser agent** | Needs JS execution to render content |
| Multi-site research across many sources | **Browser-researcher agent** | Coordinates browsing across tabs and sites |

The rule of thumb: if you can `curl` a URL and see the content you need, `web_fetch` will work. If you'd need to open a real browser and wait for things to load, you need a browser agent.

## Tips

**Search specifically.** Include version numbers, exact error messages, and library names. `"FastAPI 0.115 middleware CORS"` beats `"FastAPI middleware"` every time.

**Fetch to file for anything large.** Documentation pages, API references, and long articles should go to `/tmp/` so you can read them in chunks. This avoids truncation and keeps your conversation context lean.

**Use raw GitHub URLs.** When you need a file from a public repo, `https://raw.githubusercontent.com/{owner}/{repo}/{branch}/{path}` gives you clean text without any HTML wrapping.

**Redirects are handled automatically.** If a URL redirects (HTTP 301/302), `web_fetch` follows it. You don't need to worry about resolving shortened URLs or moved pages.

**Search first, fetch second.** Don't guess at URLs. A quick `web_search` finds the current, correct URL — documentation sites restructure constantly, and a search takes less time than debugging a 404.

**Combine with local tools.** The real power of web tools shows when you pair them with file operations: fetch documentation, save it, grep through it for the function signature you need, then use that to write code. Web tools gather context; other tools act on it.

## Next Steps

- See the [Bash Tool](./bash.md) for running commands — including `curl` when you need lower-level HTTP control
- Learn about [Filesystem Tools](./filesystem.md) for reading the files you save with `web_fetch`
- Explore [Search Tools](./search.md) for finding content in local files after you've fetched them from the web
