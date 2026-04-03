---
name: mcp-builder
description: Build Model Context Protocol (MCP) servers — research, implement, review, and evaluate MCP tools in Python/FastMCP or Node/TypeScript
origin: ComposioHQ/awesome-claude-skills
tools: Read, Write, Edit, Bash, WebSearch, WebFetch
---

# MCP Builder Skill

Design and implement MCP servers that expose tools, resources, and prompts to Claude. Emphasizes agent-centric design over simple API wrapping.

## When to Activate

- Building a new MCP server from scratch
- Wrapping an existing API as an MCP tool
- Reviewing or refactoring an existing MCP server
- Writing evaluations for MCP tool quality

## Four-Phase Process

### Phase 1 — Research & Planning
- Understand the target domain and what actions are genuinely useful to an agent
- Map capabilities to MCP primitives: Tools (actions), Resources (data), Prompts (templates)
- Design tool signatures that are self-describing

### Phase 2 — Implementation

**Python / FastMCP**
```python
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("my-server")

@mcp.tool()
def search_docs(query: str, limit: int = 10) -> list[dict]:
    """Search documentation by keyword. Returns list of {title, url, snippet}."""
    # implementation
    ...

if __name__ == "__main__":
    mcp.run()
```

**TypeScript / Node**
```typescript
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";

const server = new McpServer({ name: "my-server", version: "1.0.0" });

server.tool("search_docs", { query: z.string(), limit: z.number().default(10) },
  async ({ query, limit }) => ({
    content: [{ type: "text", text: JSON.stringify(await searchDocs(query, limit)) }]
  })
);

const transport = new StdioServerTransport();
await server.connect(transport);
```

### Phase 3 — Review & Refinement
- Tool names should be verbs describing the action
- Descriptions must explain inputs, outputs, and when to use the tool
- Error messages should guide the agent toward correct usage
- Avoid tools that require the agent to know internal IDs it can't discover

### Phase 4 — Evaluations
```python
# Test each tool with realistic agent inputs
def test_search_docs():
    result = search_docs("authentication flow", limit=5)
    assert len(result) <= 5
    assert all("url" in r for r in result)
```

## Design Principles

- **Agent-centric, not API-centric** — expose what an agent needs, not what the API exposes
- **Self-describing** — tool + parameter descriptions should be sufficient without external docs
- **Idempotent where possible** — safe to retry on failure
- **Fail informatively** — errors should explain what went wrong and what to try instead

## Links

- [MCP Python SDK](https://github.com/modelcontextprotocol/python-sdk)
- [MCP TypeScript SDK](https://github.com/modelcontextprotocol/typescript-sdk)
- [MCP specification](https://modelcontextprotocol.io)
