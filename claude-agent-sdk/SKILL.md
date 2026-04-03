---
name: claude-agent-sdk
description: Build autonomous agents with the Claude Agent SDK — research agents, chief-of-staff, observability agents, site reliability agents, and multi-agent subagent patterns
origin: anthropics/claude-cookbooks/claude_agent_sdk
tools: Read, Write, Edit, Bash
---

# Claude Agent SDK Skill

Build production-grade autonomous agents using the Claude Agent SDK. Agents adapt their strategy dynamically, use tools, spawn subagents, and handle long-running tasks.

## When to Activate

- Building an autonomous research or analysis agent
- Creating a multi-step workflow that adapts based on intermediate results
- Implementing subagent architectures (agents that spawn other agents)
- Migrating from OpenAI Agents SDK to Claude

## Requirements

```bash
pip install claude-agent-sdk python-dotenv
# Python 3.11+ required (async/await throughout)
export ANTHROPIC_API_KEY=your_key
```

## The One-Liner Research Agent

```python
import asyncio
from claude_agent_sdk import run_agent

async def main():
    result = await run_agent(
        prompt="Research the latest developments in quantum computing and summarize the top 3 breakthroughs",
        model="claude-opus-4-6",
        tools=["web_search"]
    )
    print(result)

asyncio.run(main())
```

## Full Agent with Custom Tools

```python
import asyncio
from claude_agent_sdk import Agent, Tool, run

async def search_web(query: str) -> str:
    """Search the web for current information."""
    # Your search implementation
    return f"Search results for: {query}"

async def save_file(filename: str, content: str) -> str:
    """Save content to a file."""
    with open(filename, "w") as f:
        f.write(content)
    return f"Saved to {filename}"

agent = Agent(
    model="claude-opus-4-6",
    system_prompt="""You are a research agent. When given a topic:
1. Search for current information
2. Synthesize findings
3. Save a structured report
Adapt your strategy based on what you find.""",
    tools=[
        Tool(name="search_web", func=search_web, description="Search the web"),
        Tool(name="save_file", func=save_file, description="Save content to a file"),
    ],
    max_turns=20
)

async def main():
    result = await run(agent, "Research AI safety developments in 2025")
    print(result.final_message)

asyncio.run(main())
```

## Multi-Agent: Subagent Pattern

```python
from claude_agent_sdk import Agent, SubagentTool

researcher = Agent(model="claude-opus-4-6", system_prompt="You are a researcher...")
writer = Agent(model="claude-sonnet-4-6", system_prompt="You are a technical writer...")

chief_of_staff = Agent(
    model="claude-opus-4-6",
    system_prompt="Coordinate research and writing tasks using your subagents.",
    tools=[
        SubagentTool(name="researcher", agent=researcher),
        SubagentTool(name="writer", agent=writer),
    ]
)
```

## Observability Agent Pattern

```python
from claude_agent_sdk import Agent, Tool
import json

class ObservabilityAgent:
    def __init__(self):
        self.traces = []

    async def check_metrics(self, service: str) -> dict:
        # Query your monitoring system
        return {"p99_latency_ms": 450, "error_rate": 0.02, "service": service}

    async def run(self, alert: dict) -> str:
        agent = Agent(
            model="claude-sonnet-4-6",
            system_prompt="You are an SRE agent. Diagnose alerts and recommend actions.",
            tools=[Tool(name="check_metrics", func=self.check_metrics)]
        )
        return await run(agent, f"Alert: {json.dumps(alert)}")
```

## Session Browser (Stateful Agent)

```python
from claude_agent_sdk import Agent, SessionBrowser

# Agent that maintains browser state across turns
browser_agent = Agent(
    model="claude-sonnet-4-6",
    system_prompt="You are a web automation agent.",
    tools=[SessionBrowser()]  # persistent browser session
)
```

## Migrating from OpenAI Agents SDK

```python
# OpenAI Agents pattern:
# from openai import OpenAI
# client = OpenAI()
# thread = client.beta.threads.create()
# run = client.beta.threads.runs.create(thread_id=thread.id, assistant_id=assistant.id)

# Claude Agent SDK equivalent:
from claude_agent_sdk import Agent, run
agent = Agent(model="claude-opus-4-6", system_prompt="...", tools=[...])
result = await run(agent, user_message)
```

## Links

- [Cookbook: claude_agent_sdk/](https://github.com/anthropics/claude-cookbooks/tree/main/claude_agent_sdk)
- [Claude Agent SDK docs](https://docs.anthropic.com/en/docs/claude-agent-sdk)
