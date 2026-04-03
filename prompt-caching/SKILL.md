---
name: prompt-caching
description: Cut costs up to 90% and latency by 2x with Claude prompt caching — cache system prompts, large documents, tool definitions, and conversation history
origin: anthropics/claude-cookbooks/misc/prompt_caching.ipynb
tools: Read, Write, Edit, Bash
---

# Prompt Caching Skill

Dramatically reduce cost and latency by caching repeated content (large system prompts, documents, tool schemas) across API calls. Cache hits cost 90% less than cache misses.

## When to Activate

- Sending the same large document or system prompt repeatedly
- Building a chat interface with a growing conversation history
- Using many tools whose schemas don't change between calls
- Processing a document with multiple follow-up questions

## How Caching Works

Mark content blocks with `"cache_control": {"type": "ephemeral"}`. Anthropic caches everything up to and including that block. Subsequent calls with the same prefix hit the cache.

**Pricing**: cache write = 1.25x base price | cache read = 0.1x base price (90% savings)  
**Cache TTL**: 5 minutes (refreshed on each cache hit)

## Basic: Cache a System Prompt

```python
import anthropic

client = anthropic.Anthropic()

def ask_with_cached_system(question: str) -> str:
    response = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=1024,
        system=[
            {
                "type": "text",
                "text": "You are an expert assistant with deep knowledge of...\n" + very_long_context,
                "cache_control": {"type": "ephemeral"}   # cache everything up to here
            }
        ],
        messages=[{"role": "user", "content": question}]
    )
    return response.content[0].text
```

## Cache a Large Document + Multiple Questions

```python
def ask_about_document(document_text: str, questions: list[str]) -> list[str]:
    """Ask multiple questions about a document — only pay to cache once."""
    answers = []
    for question in questions:
        response = client.messages.create(
            model="claude-sonnet-4-6",
            max_tokens=1024,
            messages=[
                {
                    "role": "user",
                    "content": [
                        {
                            "type": "text",
                            "text": f"Document:\n\n{document_text}",
                            "cache_control": {"type": "ephemeral"}  # cache the doc
                        },
                        {
                            "type": "text",
                            "text": f"\nQuestion: {question}"       # this part changes
                        }
                    ]
                }
            ]
        )
        answers.append(response.content[0].text)
    return answers
```

## Cache Conversation History

```python
def chat_with_cache(conversation: list[dict], new_message: str) -> str:
    """Cache the conversation history so far."""
    messages = []

    # Mark all but the last turn as cacheable
    for i, msg in enumerate(conversation):
        if i == len(conversation) - 1:
            # Add cache_control to the last cached turn
            content = msg["content"]
            if isinstance(content, str):
                content = [{"type": "text", "text": content, "cache_control": {"type": "ephemeral"}}]
            messages.append({"role": msg["role"], "content": content})
        else:
            messages.append(msg)

    messages.append({"role": "user", "content": new_message})

    response = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=1024,
        messages=messages
    )
    return response.content[0].text
```

## Cache Tool Definitions

```python
response = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    tools=[
        {
            "name": "search",
            "description": "Search the web...",
            "input_schema": {...},
            "cache_control": {"type": "ephemeral"}  # cache all tools up to here
        },
        # more tools...
    ],
    messages=messages
)
```

## Monitor Cache Usage

```python
def print_usage(response):
    usage = response.usage
    print(f"Input tokens:         {usage.input_tokens}")
    print(f"Cache write tokens:   {getattr(usage, 'cache_creation_input_tokens', 0)}")
    print(f"Cache read tokens:    {getattr(usage, 'cache_read_input_tokens', 0)}")
    print(f"Output tokens:        {usage.output_tokens}")

    # Estimate cost savings
    cache_reads = getattr(usage, 'cache_read_input_tokens', 0)
    savings = cache_reads * 0.9  # 90% cheaper than uncached
    print(f"Approx tokens saved:  {savings:.0f}")
```

## Minimum Cache Size

Content must be at least **1024 tokens** to be eligible for caching. Smaller blocks won't be cached even with `cache_control` set.

## Speculative Caching (Beta)

Pre-populate the cache before the user's first request:

```python
# Send a prefill request with no user message to warm the cache
client.messages.create(
    model="claude-sonnet-4-6", max_tokens=1,
    system=[{"type": "text", "text": large_system_prompt, "cache_control": {"type": "ephemeral"}}],
    messages=[{"role": "user", "content": "Hi"}]  # dummy message to trigger cache write
)
```

## Links

- [Cookbook: misc/prompt_caching.ipynb](https://github.com/anthropics/claude-cookbooks/blob/main/misc/prompt_caching.ipynb)
- [Prompt caching docs](https://docs.anthropic.com/en/docs/build-with-claude/prompt-caching)
