---
name: memory-management
description: Implement memory and context management for long-running Claude agents — in-context, external, session compaction, and the memory tool pattern
origin: anthropics/claude-cookbooks/tool_use/memory_cookbook.ipynb
tools: Read, Write, Edit, Bash
---

# Memory Management Skill

Give Claude agents persistent memory across sessions and manage context windows for long-running tasks. Covers in-context memory, external storage, session compaction, and the memory tool.

## When to Activate

- Building a chatbot that remembers user preferences across sessions
- Agent accumulating context that exceeds the context window
- Storing and retrieving facts learned during a task
- Implementing session continuity for long-running workflows

## Memory Types

| Type | Scope | Best For |
|------|-------|----------|
| In-context | Current session only | Short conversations |
| External (DB/file) | Persistent | Cross-session facts |
| Session compaction | Current session | Long conversations |
| Memory tool | Agent-controlled | Autonomous agents |

## 1. In-Context Memory (Conversation History)

```python
import anthropic

client = anthropic.Anthropic()

class ConversationMemory:
    def __init__(self, system_prompt: str = ""):
        self.messages = []
        self.system = system_prompt

    def chat(self, user_input: str) -> str:
        self.messages.append({"role": "user", "content": user_input})
        response = client.messages.create(
            model="claude-sonnet-4-6",
            max_tokens=1024,
            system=self.system,
            messages=self.messages
        )
        assistant_reply = response.content[0].text
        self.messages.append({"role": "assistant", "content": assistant_reply})
        return assistant_reply
```

## 2. External Memory (File-Based)

```python
import json
from pathlib import Path
from datetime import datetime

class PersistentMemory:
    def __init__(self, memory_file: str = "agent_memory.json"):
        self.path = Path(memory_file)
        self.memory = json.loads(self.path.read_text()) if self.path.exists() else {}

    def store(self, key: str, value: str, category: str = "general"):
        self.memory[key] = {"value": value, "category": category, "updated": datetime.now().isoformat()}
        self.path.write_text(json.dumps(self.memory, indent=2))

    def retrieve(self, key: str) -> str | None:
        return self.memory.get(key, {}).get("value")

    def search(self, category: str) -> dict:
        return {k: v["value"] for k, v in self.memory.items() if v.get("category") == category}

    def as_context(self) -> str:
        if not self.memory:
            return ""
        lines = [f"- {k}: {v['value']}" for k, v in self.memory.items()]
        return "Known facts:\n" + "\n".join(lines)
```

## 3. Memory Tool (Agent-Controlled)

Let Claude decide what to remember by giving it a memory tool:

```python
memory_store = {}

memory_tools = [
    {
        "name": "remember",
        "description": "Store a fact or piece of information for later retrieval.",
        "input_schema": {
            "type": "object",
            "properties": {
                "key": {"type": "string", "description": "Unique identifier for this memory"},
                "value": {"type": "string", "description": "The information to remember"},
                "importance": {"type": "string", "enum": ["low", "medium", "high"]}
            },
            "required": ["key", "value"]
        }
    },
    {
        "name": "recall",
        "description": "Retrieve a previously stored memory by key.",
        "input_schema": {
            "type": "object",
            "properties": {"key": {"type": "string"}},
            "required": ["key"]
        }
    }
]

def handle_memory_tool(tool_name: str, tool_input: dict) -> str:
    if tool_name == "remember":
        memory_store[tool_input["key"]] = tool_input["value"]
        return f"Stored: {tool_input['key']}"
    elif tool_name == "recall":
        return memory_store.get(tool_input["key"], "Not found")
```

## 4. Session Compaction (Context Window Management)

When conversation history grows too long, summarize and compress:

```python
def compact_session(messages: list[dict], max_tokens: int = 50000) -> list[dict]:
    """Summarize old messages to free context window space."""
    # Estimate token count (rough: 1 token ≈ 4 chars)
    total_chars = sum(len(str(m["content"])) for m in messages)
    if total_chars / 4 < max_tokens:
        return messages  # no compaction needed

    # Keep the last N messages, summarize the rest
    keep_recent = 10
    to_summarize = messages[:-keep_recent]
    recent = messages[-keep_recent:]

    summary_response = client.messages.create(
        model="claude-haiku-4-5",
        max_tokens=500,
        messages=[
            {"role": "user", "content": f"Summarize this conversation concisely, preserving key facts:\n\n{str(to_summarize)}"}
        ]
    )
    summary = summary_response.content[0].text

    return [
        {"role": "user", "content": f"[Previous conversation summary: {summary}]"},
        {"role": "assistant", "content": "Understood. I have context from our previous exchange."},
        *recent
    ]
```

## 5. Automatic Context Compaction

Use the `automatic-context-compaction` cookbook pattern — monitor token usage and trigger compaction automatically:

```python
def chat_with_auto_compaction(memory: ConversationMemory, user_input: str) -> str:
    # Check if we're approaching context limit
    response = memory.chat(user_input)

    # After response, check usage
    if len(memory.messages) > 40:  # rough heuristic
        memory.messages = compact_session(memory.messages)

    return response
```

## Links

- [Cookbook: tool_use/memory_cookbook.ipynb](https://github.com/anthropics/claude-cookbooks/blob/main/tool_use/memory_cookbook.ipynb)
- [Cookbook: misc/session_memory_compaction.ipynb](https://github.com/anthropics/claude-cookbooks/blob/main/misc/session_memory_compaction.ipynb)
- [Context window docs](https://docs.anthropic.com/en/docs/build-with-claude/context-windows)
