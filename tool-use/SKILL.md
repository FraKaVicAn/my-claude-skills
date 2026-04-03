---
name: tool-use
description: Implement Claude tool use — define tools, handle tool calls, run parallel tools, use tool_choice, and build tool-calling loops with the Anthropic SDK
origin: anthropics/claude-cookbooks/tool_use
tools: Read, Write, Edit, Bash
---

# Tool Use Skill

Give Claude the ability to call functions, APIs, and external services. Handle single tools, parallel tool calls, forced tool choice, and multi-turn tool loops.

## When to Activate

- Giving Claude access to search, calculators, databases, or APIs
- Extracting structured data using tool schemas as the output format
- Building agents that take real-world actions
- Implementing parallel tool execution for speed

## Basic Tool Definition

```python
import anthropic
import json

client = anthropic.Anthropic()

tools = [
    {
        "name": "get_weather",
        "description": "Get current weather for a city. Returns temperature (°C), conditions, and humidity.",
        "input_schema": {
            "type": "object",
            "properties": {
                "city": {"type": "string", "description": "City name, e.g. 'Paris'"},
                "units": {"type": "string", "enum": ["celsius", "fahrenheit"], "default": "celsius"}
            },
            "required": ["city"]
        }
    }
]

def get_weather(city: str, units: str = "celsius") -> dict:
    # Your actual implementation
    return {"temp": 18, "conditions": "Partly cloudy", "humidity": 65}
```

## Tool Calling Loop (Standard Pattern)

```python
def run_with_tools(user_message: str) -> str:
    messages = [{"role": "user", "content": user_message}]

    while True:
        response = client.messages.create(
            model="claude-sonnet-4-6",
            max_tokens=1024,
            tools=tools,
            messages=messages
        )

        if response.stop_reason == "end_turn":
            return next(b.text for b in response.content if b.type == "text")

        if response.stop_reason == "tool_use":
            # Add assistant response to history
            messages.append({"role": "assistant", "content": response.content})

            # Execute all tool calls
            tool_results = []
            for block in response.content:
                if block.type == "tool_use":
                    result = get_weather(**block.input)  # dispatch to right function
                    tool_results.append({
                        "type": "tool_result",
                        "tool_use_id": block.id,
                        "content": json.dumps(result)
                    })

            messages.append({"role": "user", "content": tool_results})

result = run_with_tools("What's the weather like in Tokyo and Paris?")
```

## Parallel Tool Calls

Claude will automatically call multiple tools in one turn when appropriate. Handle all `tool_use` blocks before sending results back — the pattern above already handles this.

## Forced Tool Choice

```python
# Force Claude to always use a specific tool
response = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    tools=tools,
    tool_choice={"type": "tool", "name": "get_weather"},  # force this tool
    messages=messages
)

# Or require any tool (not text)
tool_choice={"type": "any"}

# Or let Claude decide (default)
tool_choice={"type": "auto"}
```

## Tool Use with Pydantic

```python
from pydantic import BaseModel
import json

class WeatherInput(BaseModel):
    city: str
    units: str = "celsius"

# Validate and parse tool input safely
for block in response.content:
    if block.type == "tool_use" and block.name == "get_weather":
        validated = WeatherInput(**block.input)
        result = get_weather(validated.city, validated.units)
```

## Vision with Tools

```python
import base64

with open("chart.png", "rb") as f:
    image_data = base64.b64encode(f.read()).decode()

response = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    tools=tools,
    messages=[{
        "role": "user",
        "content": [
            {"type": "image", "source": {"type": "base64", "media_type": "image/png", "data": image_data}},
            {"type": "text", "text": "What city is shown in this weather chart? Get its current weather."}
        ]
    }]
)
```

## Best Practices

- Write tool descriptions as if explaining to a smart colleague — inputs, outputs, when to use
- Return structured data (JSON) from tool functions, not prose
- Include error information in tool results rather than raising exceptions
- For expensive tools, implement caching on the tool side
- Keep tool schemas minimal — only required fields reduce token usage

## Links

- [Cookbook: tool_use/](https://github.com/anthropics/claude-cookbooks/tree/main/tool_use)
- [Tool use docs](https://docs.anthropic.com/en/docs/build-with-claude/tool-use)
