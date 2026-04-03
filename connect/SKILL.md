---
name: connect
description: Connect Claude to external applications via Composio MCP tool routing — OAuth handled automatically, 1000+ services supported
origin: ComposioHQ/awesome-claude-skills
tools: Bash, Read, Write
---

# Connect Skill

Route Claude's tool calls through Composio to take real actions across Gmail, Slack, GitHub, Notion, and 1000+ services. Uses the MCP (Model Context Protocol) server approach.

## When to Activate

- Any time Claude needs to perform an action in an external app
- Automating workflows that span multiple services
- Building agents that interact with real-world systems

## Setup

```bash
# Python
pip install composio-core composio-mcp
export COMPOSIO_API_KEY=your_key

# TypeScript
npm install composio-core
export COMPOSIO_API_KEY=your_key
```

## Supported Frameworks

- Claude Agent SDK
- OpenAI Agents
- Vercel AI SDK
- LangChain / LangGraph
- Any MCP-compatible client

## Authentication Flow

First time using an app:
1. Claude provides an authorization link
2. Click and confirm access
3. Connection persists — no re-auth needed

## Links

- [platform.composio.dev](https://platform.composio.dev)
- [github.com/ComposioHQ/awesome-claude-skills](https://github.com/ComposioHQ/awesome-claude-skills)
