---
name: extended-thinking
description: Use Claude's extended thinking for complex reasoning — set budget_tokens, handle thinking blocks, stream thinking output, and combine with tool use
origin: anthropics/claude-cookbooks/extended_thinking
tools: Read, Write, Edit, Bash
---

# Extended Thinking Skill

Enable Claude's extended thinking for problems requiring deep reasoning: math, logic, multi-step planning, and complex analysis. Claude shows its work in `thinking` blocks before answering.

## When to Activate

- Complex math or logic problems
- Multi-step reasoning or planning tasks
- Problems where you want to see Claude's reasoning process
- Tasks where accuracy matters more than latency
- Difficult coding or algorithm design questions

## Basic Usage

```python
import anthropic

client = anthropic.Anthropic()

response = client.messages.create(
    model="claude-sonnet-4-6",   # extended thinking supported on Sonnet+
    max_tokens=16000,
    thinking={
        "type": "enabled",
        "budget_tokens": 10000   # how many tokens Claude can use for thinking
    },
    messages=[{
        "role": "user",
        "content": "Solve this step by step: If a hotel bill of $30 is split 3 ways, and each person gets $1 back, where did the missing dollar go?"
    }]
)

# Response contains both thinking and text blocks
for block in response.content:
    if block.type == "thinking":
        print("THINKING:", block.thinking[:500], "...")
    elif block.type == "text":
        print("ANSWER:", block.text)
```

## Helper: Pretty-Print Thinking

```python
def print_thinking_response(response):
    for i, block in enumerate(response.content):
        if block.type == "thinking":
            print(f"{'='*60}")
            print(f"THINKING BLOCK {i+1}")
            print(f"{'='*60}")
            print(block.thinking)
            print()
        elif block.type == "text":
            print(f"{'='*60}")
            print("FINAL ANSWER")
            print(f"{'='*60}")
            print(block.text)
```

## Streaming Extended Thinking

```python
with client.messages.stream(
    model="claude-sonnet-4-6",
    max_tokens=16000,
    thinking={"type": "enabled", "budget_tokens": 8000},
    messages=[{"role": "user", "content": "Design an algorithm for..."}]
) as stream:
    for event in stream:
        if hasattr(event, "type"):
            if event.type == "content_block_start":
                if hasattr(event.content_block, "type"):
                    if event.content_block.type == "thinking":
                        print("Claude is thinking...")
            elif event.type == "content_block_delta":
                if hasattr(event.delta, "thinking"):
                    print(event.delta.thinking, end="", flush=True)
                elif hasattr(event.delta, "text"):
                    print(event.delta.text, end="", flush=True)
```

## Extended Thinking + Tool Use

```python
response = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=16000,
    thinking={"type": "enabled", "budget_tokens": 5000},
    tools=tools,
    messages=messages
)
# Claude will think before deciding which tools to call
```

## Budget Tokens Guide

| Task Complexity | Suggested budget_tokens |
|----------------|------------------------|
| Simple reasoning | 1,000 – 3,000 |
| Medium complexity | 3,000 – 8,000 |
| Hard math/logic | 8,000 – 20,000 |
| Very complex planning | 20,000 – 32,000 |

- `budget_tokens` sets the *maximum* — Claude uses only what it needs
- `max_tokens` must be > `budget_tokens` + expected answer length
- Redacted thinking blocks may appear for sensitive content — this is normal

## Redacted Thinking Blocks

```python
for block in response.content:
    if block.type == "redacted_thinking":
        print("(thinking redacted for safety)")
    elif block.type == "thinking":
        print("Thinking:", block.thinking)
```

## Links

- [Cookbook: extended_thinking/](https://github.com/anthropics/claude-cookbooks/tree/main/extended_thinking)
- [Extended thinking docs](https://docs.anthropic.com/en/docs/build-with-claude/extended-thinking)
