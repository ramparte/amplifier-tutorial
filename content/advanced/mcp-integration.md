---
id: mcp-integration
type: advanced
title: "MCP Integration"
---

# MCP Integration

Connect Amplifier to Model Context Protocol servers.

## Overview

MCP (Model Context Protocol) is a standard for connecting AI assistants to external tools and data sources. Amplifier supports MCP servers, letting you:

- **Connect to databases** - Query and modify data
- **Access APIs** - Any service with MCP support
- **Use external tools** - Capabilities from MCP ecosystem
- **Share tools** - Same server works with multiple clients

## What is MCP?

MCP separates:
- **Client** - The AI assistant (Amplifier)
- **Server** - Provider of tools/resources

```
Amplifier (Client) ←→ MCP Server ←→ External System
```

Benefits:
- Standard protocol
- Reusable servers
- Ecosystem of tools
- Works with multiple AI clients

## Configuring MCP Servers

### In Settings

```yaml
# ~/.amplifier/settings.yaml
mcp:
  servers:
    - name: filesystem
      command: npx @modelcontextprotocol/server-filesystem
      args: ["/allowed/path"]
      
    - name: postgres
      command: npx @modelcontextprotocol/server-postgres
      env:
        DATABASE_URL: "postgresql://user:pass@localhost/db"
```

### In Bundle

```yaml
# bundle.yaml
mcp:
  servers:
    - name: my-server
      command: python my_server.py
```

## Available MCP Servers

### Official Servers

| Server | Purpose |
|--------|---------|
| `@modelcontextprotocol/server-filesystem` | File operations |
| `@modelcontextprotocol/server-postgres` | PostgreSQL queries |
| `@modelcontextprotocol/server-sqlite` | SQLite databases |
| `@modelcontextprotocol/server-github` | GitHub API |

### Installing Servers

```bash
# Via npm
npm install -g @modelcontextprotocol/server-filesystem
npm install -g @modelcontextprotocol/server-postgres

# Via pip
pip install mcp-server-fetch
```

## Example: Filesystem Server

### Configure

```yaml
mcp:
  servers:
    - name: files
      command: npx @modelcontextprotocol/server-filesystem
      args: ["/home/user/projects"]
```

### Use

The server adds tools that Amplifier can use:

```
> List files in /home/user/projects
> Read the contents of package.json
```

## Example: Database Server

### Configure

```yaml
mcp:
  servers:
    - name: database
      command: npx @modelcontextprotocol/server-postgres
      env:
        DATABASE_URL: "postgresql://localhost/mydb"
```

### Use

```
> Show all tables in the database
> Query the users table for active users
> Count orders from last week
```

## Example: GitHub Server

### Configure

```yaml
mcp:
  servers:
    - name: github
      command: npx @modelcontextprotocol/server-github
      env:
        GITHUB_TOKEN: ${GITHUB_TOKEN}
```

### Use

```
> List open PRs in microsoft/amplifier
> Get details of issue #123
> Show my recent commits
```

## Creating MCP Servers

### Python Server

```python
# my_server.py
from mcp.server import Server
from mcp.types import Tool, TextContent

server = Server("my-server")

@server.tool()
async def get_weather(city: str) -> str:
    """Get weather for a city"""
    # Implementation
    return f"Weather in {city}: Sunny, 72°F"

@server.tool()
async def search_docs(query: str) -> str:
    """Search internal documentation"""
    # Implementation
    return f"Found 5 results for: {query}"

if __name__ == "__main__":
    server.run()
```

### Register

```yaml
mcp:
  servers:
    - name: my-tools
      command: python my_server.py
```

## MCP Resources

MCP servers can also provide resources (data):

```python
@server.resource("docs://{path}")
async def get_doc(path: str) -> str:
    """Get documentation by path"""
    with open(f"/docs/{path}") as f:
        return f.read()
```

Access in Amplifier:
```
> Get the resource docs://api/auth.md
```

## Debugging MCP

### Test Server Directly

```bash
# Run server
npx @modelcontextprotocol/server-filesystem /tmp

# In another terminal, test with mcp CLI
mcp test-server filesystem
```

### Check Logs

```bash
# Amplifier logs MCP communication
export AMPLIFIER_LOG_LEVEL=debug
amp
```

### Common Issues

**Server not starting:**
- Check command path
- Verify dependencies installed
- Check environment variables

**Tools not appearing:**
- Restart Amplifier
- Verify server is running
- Check server logs

**Permission errors:**
- Check allowed paths
- Verify credentials
- Check environment setup

## Security Considerations

### Principle of Least Privilege

```yaml
mcp:
  servers:
    # Good - limited scope
    - name: project-files
      command: npx @modelcontextprotocol/server-filesystem
      args: ["/home/user/current-project"]
      
    # Avoid - too broad
    - name: all-files
      command: npx @modelcontextprotocol/server-filesystem
      args: ["/"]  # Don't do this!
```

### Credential Management

```yaml
mcp:
  servers:
    - name: database
      command: npx @modelcontextprotocol/server-postgres
      env:
        # Use environment variable, not hardcoded
        DATABASE_URL: ${DATABASE_URL}
```

### Audit MCP Actions

MCP tool calls appear in session logs:

```bash
cat events.jsonl | jq 'select(.tool_name | startswith("mcp:"))'
```

## Try It Yourself

### Exercise 1: Filesystem Server

```yaml
# Add to ~/.amplifier/settings.yaml
mcp:
  servers:
    - name: files
      command: npx @modelcontextprotocol/server-filesystem
      args: ["./"]
```

```bash
npm install -g @modelcontextprotocol/server-filesystem
amp

> What files are in this directory?
```

### Exercise 2: SQLite Server

```yaml
mcp:
  servers:
    - name: db
      command: npx @modelcontextprotocol/server-sqlite
      args: ["./test.db"]
```

```
> Create a users table
> Insert some test data
> Query all users
```

## Resources

- [MCP Specification](https://modelcontextprotocol.io)
- [Official Servers](https://github.com/modelcontextprotocol/servers)
- [Python SDK](https://github.com/modelcontextprotocol/python-sdk)
- [TypeScript SDK](https://github.com/modelcontextprotocol/typescript-sdk)
