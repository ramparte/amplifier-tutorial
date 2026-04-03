---
id: mcp
type: app
title: "MCP Integration"
---

# MCP Integration

Amplifier's built-in tools -- bash, read_file, grep, web_search -- cover a lot of ground. But sometimes you need capabilities that don't ship with any bundle: querying a Postgres database, calling a company-internal API, searching a vector store, or controlling a Kubernetes cluster. That's where MCP comes in.

MCP (Model Context Protocol) is an open standard for connecting AI assistants to external tool servers. Instead of writing a custom Amplifier module for every new capability, you point Amplifier at an MCP server, and its tools appear alongside the native ones. The LLM doesn't know the difference -- it sees `query_database` next to `read_file` and picks whichever one fits the task.

## How MCP Works

The architecture is straightforward:

```
Amplifier (MCP Client)
    |
    v
MCP Server (separate process)
    |
    v
External System (database, API, service)
```

Amplifier acts as an **MCP client**. It connects to one or more **MCP servers**, discovers what tools they offer, and makes those tools available to the LLM during a session. Each MCP server is a standalone process -- it might be an npm package, a Python script, or a compiled binary. The server handles the actual communication with whatever external system it wraps.

The protocol is standardized: tool discovery, invocation, and response all follow the MCP spec. This means any MCP server works with any MCP client. A server you build for Amplifier also works with other MCP-compatible tools, and servers built for other clients work with Amplifier.

## Configuring MCP Servers

You can declare MCP servers in two places: your user settings (available in every session) or a specific bundle (available only when that bundle is loaded).

### In User Settings

For servers you want available everywhere:

```yaml
# ~/.amplifier/settings.yaml
mcp:
  servers:
    - name: postgres
      command: npx @modelcontextprotocol/server-postgres
      env:
        DATABASE_URL: ${DATABASE_URL}

    - name: github
      command: npx @modelcontextprotocol/server-github
      env:
        GITHUB_TOKEN: ${GITHUB_TOKEN}
```

These servers start automatically when a session begins. Environment variables are resolved at launch, so you can use `${VAR}` references to keep credentials out of config files.

### In a Bundle

For servers tied to a specific project or workflow:

```yaml
# my-bundle/bundle.yaml
bundle:
  name: data-analyst
  version: 1.0.0

includes:
  - bundle: foundation

mcp:
  servers:
    - name: warehouse
      command: python ./mcp_servers/warehouse_server.py
      args: ["--read-only"]

    - name: analytics
      command: npx @company/analytics-mcp
      env:
        API_KEY: ${ANALYTICS_KEY}
```

Bundle-level servers only run when that bundle is active. This keeps project-specific tools from cluttering unrelated sessions.

## How MCP Tools Appear

Once configured, MCP tools show up alongside native tools seamlessly. The LLM sees a unified tool list and picks the right one based on your request.

> How many active users signed up last week?

```
[Tool: mcp:warehouse/query] SELECT COUNT(*) FROM users
    WHERE created_at > NOW() - INTERVAL '7 days' AND status = 'active'
    Result: 1,247 new active users in the last 7 days.
```

The `mcp:warehouse/query` prefix tells you the tool came from the MCP server named `warehouse`. But to the LLM, it's just another tool. It made the same kind of decision it makes when picking `grep` over `read_file` -- matching the tool's description to your intent.

> What open issues are assigned to me?

```
[Tool: mcp:github/list_issues] owner="acme" repo="backend" assignee="@me" state="open"
    Found 3 open issues:
    #421 - Fix rate limiter edge case
    #418 - Add pagination to /users endpoint
    #415 - Update API docs for v2
```

Native and MCP tools can work together in the same turn. The LLM might use `grep` to find something in your codebase, then call an MCP tool to check the production database, then use `edit_file` to apply a fix -- all in one response.

## Use Cases

MCP integration shines in several scenarios:

**Databases** -- Connect to Postgres, SQLite, or any database without writing custom tools. The MCP server handles queries and returns results. The LLM can explore schemas, write queries, and summarize data.

**APIs and services** -- Wrap any REST or GraphQL API in an MCP server. Company-internal services, SaaS platforms, monitoring systems -- anything with an API becomes a tool the LLM can use.

**Specialized tools** -- Vector search, code analysis, infrastructure management, CI/CD systems. The MCP ecosystem has servers for many of these, and writing new ones is straightforward.

**Cross-tool workflows** -- Because MCP tools appear alongside native tools, the LLM can chain them naturally. "Find the slow endpoint in the logs, query the database for related errors, and create a GitHub issue" spans three different MCP servers plus native file tools, and the LLM orchestrates it without special configuration.

## Practical Example: Connecting a Database

Let's walk through setting up a Postgres MCP server from scratch.

### Install the Server

```bash
npm install -g @modelcontextprotocol/server-postgres
```

### Configure It

```yaml
# ~/.amplifier/settings.yaml
mcp:
  servers:
    - name: appdb
      command: npx @modelcontextprotocol/server-postgres
      env:
        DATABASE_URL: "postgresql://dev:devpass@localhost:5432/myapp"
```

### Use It

Start a session and ask questions about your data:

> What tables are in the database?

```
[Tool: mcp:appdb/list_tables]
    Tables: users, orders, products, order_items, sessions
```

> Show me the top 10 customers by order count

```
[Tool: mcp:appdb/query] SELECT u.name, COUNT(o.id) as order_count
    FROM users u JOIN orders o ON u.id = o.user_id
    GROUP BY u.name ORDER BY order_count DESC LIMIT 10
    ...
```

> Are there any orders stuck in "processing" for more than 24 hours?

```
[Tool: mcp:appdb/query] SELECT id, user_id, created_at
    FROM orders WHERE status = 'processing'
    AND created_at < NOW() - INTERVAL '24 hours'
    Result: 3 stuck orders found. Order #8812 has been processing for 47 hours.
```

The LLM generates SQL, the MCP server executes it safely against the database, and you get answers in natural language. No custom tool module required.

## Building Your Own MCP Server

When no existing server fits your needs, you can write one in Python:

```python
from mcp.server import Server
from mcp.types import Tool, TextContent

server = Server("inventory")

@server.tool()
async def check_stock(product_id: str) -> str:
    """Check current inventory level for a product."""
    # Your business logic here
    count = await inventory_db.get_count(product_id)
    return f"Product {product_id}: {count} units in stock"

@server.tool()
async def low_stock_report() -> str:
    """Get all products with inventory below reorder threshold."""
    items = await inventory_db.get_low_stock()
    return "\n".join(f"{item.name}: {item.count} units" for item in items)

if __name__ == "__main__":
    server.run()
```

Register it in your bundle:

```yaml
mcp:
  servers:
    - name: inventory
      command: python ./mcp_servers/inventory_server.py
```

Now the LLM can check stock and pull reports:

> Are we running low on anything?

```
[Tool: mcp:inventory/low_stock_report]
    Widget A: 12 units (threshold: 50)
    Gadget X: 3 units (threshold: 25)
    Two products are below reorder threshold.
```

## Security Considerations

MCP servers run as separate processes with their own access to external systems. Keep these principles in mind:

**Least privilege.** Give each server access to only what it needs. A database server should use a read-only connection unless writes are required. A file server should be scoped to a specific directory.

**Credential hygiene.** Use environment variable references (`${DATABASE_URL}`) rather than hardcoding secrets in config files. This keeps credentials out of version control and lets you use different values per environment.

**Audit trail.** MCP tool calls appear in session event logs just like native tool calls. You can filter for them to review what external actions the LLM took:

```bash
cat events.jsonl | jq 'select(.tool_name | startswith("mcp:"))'
```

## Debugging

If an MCP server isn't working as expected, check these common issues:

**Server not starting** -- Verify the command path and that dependencies are installed. Run the command manually to see error output.

**Tools not appearing** -- Restart the Amplifier session. MCP tool discovery happens at session startup. If you added a server config after the session started, you need a new session.

**Unexpected results** -- Enable debug logging to see the raw MCP protocol messages:

```bash
export AMPLIFIER_LOG_LEVEL=debug
amp
```

This shows exactly what's being sent to and received from each MCP server.

## Next Steps

- Read [Building Your Own App](./building-apps.md) for using MCP servers in custom applications
- See [Architecture](../concepts/architecture.md) to understand how MCP fits into the module system
- Learn about [Creating Custom Tools](../advanced/custom-tool.md) for when you need a native module instead
- Explore [Creating Custom Bundles](../advanced/custom-bundle.md) for packaging MCP configurations
