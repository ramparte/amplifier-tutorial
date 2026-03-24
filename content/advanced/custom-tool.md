---
id: custom-tool
type: advanced
title: "Creating Custom Tools"
---

# Creating Custom Tools

Extend Amplifier with your own tools.

## Overview

Custom tools let you:

- **Add new capabilities** - Integrate with any system
- **Wrap existing CLIs** - Make tools AI-friendly
- **Connect to APIs** - Custom integrations
- **Automate workflows** - Complex operations as single tools

## Tool Structure

A tool is a Python module that implements the Tool protocol:

```python
# my_tool.py
from amplifier_core import Tool

class MyTool(Tool):
    name = "my-tool"
    description = "What this tool does (shown to AI)"
    input_schema = {
        "type": "object",
        "properties": {
            "param1": {
                "type": "string",
                "description": "First parameter"
            }
        },
        "required": ["param1"]
    }
    
    async def execute(self, input: dict) -> str:
        param1 = input["param1"]
        # Do something
        return f"Result: {param1}"
```

## Basic Example: Timestamp Tool

```python
# modules/timestamp_tool/tool.py
from datetime import datetime
from amplifier_core import Tool

class TimestampTool(Tool):
    name = "timestamp"
    description = "Get the current timestamp in various formats"
    input_schema = {
        "type": "object",
        "properties": {
            "format": {
                "type": "string",
                "description": "Output format: iso, unix, or human",
                "enum": ["iso", "unix", "human"],
                "default": "iso"
            }
        }
    }
    
    async def execute(self, input: dict) -> str:
        fmt = input.get("format", "iso")
        now = datetime.now()
        
        if fmt == "iso":
            return now.isoformat()
        elif fmt == "unix":
            return str(int(now.timestamp()))
        else:
            return now.strftime("%B %d, %Y at %I:%M %p")
```

## Register in Bundle

```yaml
# bundle.yaml
tools:
  - module: timestamp-tool
    source: ./modules/timestamp_tool
```

## API Integration Example

```python
# modules/weather_tool/tool.py
import httpx
from amplifier_core import Tool

class WeatherTool(Tool):
    name = "weather"
    description = "Get current weather for a location"
    input_schema = {
        "type": "object",
        "properties": {
            "city": {
                "type": "string",
                "description": "City name"
            }
        },
        "required": ["city"]
    }
    
    def __init__(self, config: dict = None):
        self.api_key = config.get("api_key") if config else None
    
    async def execute(self, input: dict) -> str:
        city = input["city"]
        
        async with httpx.AsyncClient() as client:
            response = await client.get(
                f"https://api.weather.example.com/current",
                params={"city": city, "key": self.api_key}
            )
            data = response.json()
        
        return f"Weather in {city}: {data['temp']}°F, {data['condition']}"
```

Configure with API key:

```yaml
tools:
  - module: weather-tool
    source: ./modules/weather_tool
    config:
      api_key: ${WEATHER_API_KEY}
```

## CLI Wrapper Example

Wrap an existing CLI tool:

```python
# modules/docker_tool/tool.py
import subprocess
import json
from amplifier_core import Tool

class DockerTool(Tool):
    name = "docker"
    description = "Manage Docker containers"
    input_schema = {
        "type": "object",
        "properties": {
            "action": {
                "type": "string",
                "enum": ["list", "start", "stop", "logs"],
                "description": "Action to perform"
            },
            "container": {
                "type": "string",
                "description": "Container name/ID (for start/stop/logs)"
            }
        },
        "required": ["action"]
    }
    
    async def execute(self, input: dict) -> str:
        action = input["action"]
        container = input.get("container")
        
        if action == "list":
            result = subprocess.run(
                ["docker", "ps", "--format", "json"],
                capture_output=True, text=True
            )
            return result.stdout
            
        elif action == "start":
            result = subprocess.run(
                ["docker", "start", container],
                capture_output=True, text=True
            )
            return f"Started {container}" if result.returncode == 0 else result.stderr
            
        elif action == "stop":
            result = subprocess.run(
                ["docker", "stop", container],
                capture_output=True, text=True
            )
            return f"Stopped {container}" if result.returncode == 0 else result.stderr
            
        elif action == "logs":
            result = subprocess.run(
                ["docker", "logs", "--tail", "50", container],
                capture_output=True, text=True
            )
            return result.stdout or result.stderr
```

## Input Schema

Define what the AI can pass to your tool:

```python
input_schema = {
    "type": "object",
    "properties": {
        # Required string
        "name": {
            "type": "string",
            "description": "User's name"
        },
        # Optional with default
        "count": {
            "type": "integer",
            "description": "Number of items",
            "default": 10
        },
        # Enum (limited choices)
        "format": {
            "type": "string",
            "enum": ["json", "csv", "xml"],
            "description": "Output format"
        },
        # Boolean
        "verbose": {
            "type": "boolean",
            "description": "Show detailed output",
            "default": False
        },
        # Array
        "tags": {
            "type": "array",
            "items": {"type": "string"},
            "description": "List of tags"
        }
    },
    "required": ["name"]  # Which params are required
}
```

## Error Handling

Return clear error messages:

```python
async def execute(self, input: dict) -> str:
    try:
        result = self.do_something(input)
        return result
    except FileNotFoundError as e:
        return f"Error: File not found - {e}"
    except PermissionError as e:
        return f"Error: Permission denied - {e}"
    except Exception as e:
        return f"Error: {type(e).__name__} - {e}"
```

## Async Operations

For I/O-bound operations, use async:

```python
import httpx

async def execute(self, input: dict) -> str:
    async with httpx.AsyncClient() as client:
        response = await client.get(input["url"])
        return response.text
```

## Configuration

Tools can accept configuration:

```python
class MyTool(Tool):
    def __init__(self, config: dict = None):
        config = config or {}
        self.api_key = config.get("api_key")
        self.base_url = config.get("base_url", "https://api.example.com")
```

```yaml
tools:
  - module: my-tool
    source: ./modules/my_tool
    config:
      api_key: ${API_KEY}
      base_url: https://api.custom.com
```

## Module Structure

```
modules/my_tool/
├── __init__.py      # Export the tool class
├── tool.py          # Tool implementation
└── pyproject.toml   # Dependencies (optional)
```

`__init__.py`:
```python
from .tool import MyTool
__all__ = ["MyTool"]
```

## Try It Yourself

### Exercise 1: Simple Tool

Create `modules/greet/tool.py`:

```python
from amplifier_core import Tool

class GreetTool(Tool):
    name = "greet"
    description = "Generate a greeting"
    input_schema = {
        "type": "object",
        "properties": {
            "name": {"type": "string", "description": "Name to greet"}
        },
        "required": ["name"]
    }
    
    async def execute(self, input: dict) -> str:
        return f"Hello, {input['name']}! Welcome to Amplifier."
```

Add to bundle and test:
```
> Use the greet tool to greet Alice
```

### Exercise 2: CLI Wrapper

Wrap a CLI tool you use frequently (git, kubectl, etc.).

### Exercise 3: API Integration

Create a tool that calls an API you work with.

## Best Practices

1. **Clear descriptions** - Help the AI know when to use your tool
2. **Specific schemas** - Use enums and defaults where appropriate
3. **Good error messages** - Return actionable error info
4. **Async when possible** - Don't block on I/O
5. **Minimal dependencies** - Keep tools lightweight
