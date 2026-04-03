---
name: summarization
description: Summarize documents, conversations, and web pages with Claude — extractive, abstractive, hierarchical, and multi-document summarization with prompt caching
origin: anthropics/claude-cookbooks/capabilities/summarization
tools: Read, Write, Edit, Bash
---

# Summarization Skill

Produce accurate, well-structured summaries of any length or format: bullet points, executive summaries, meeting notes, multi-document synthesis, and more.

## When to Activate

- Summarizing long documents, reports, or articles
- Condensing conversation histories or meeting transcripts
- Creating executive summaries from technical documents
- Synthesizing multiple documents on the same topic
- Building a document digest pipeline

## Basic Summarization

```python
import anthropic

client = anthropic.Anthropic()

def summarize(text: str, style: str = "concise", max_words: int = 200) -> str:
    styles = {
        "concise": "Write a concise summary in plain language.",
        "bullets": "Summarize as 5-7 bullet points covering the main ideas.",
        "executive": "Write an executive summary: key findings, recommendations, and next steps.",
        "technical": "Summarize technical details, preserving precision and specificity.",
        "eli5": "Explain the main idea simply enough for a non-expert to understand."
    }
    prompt = styles.get(style, styles["concise"])
    response = client.messages.create(
        model="claude-haiku-4-5",
        max_tokens=max_words * 2,   # generous token budget
        system=f"{prompt} Target length: ~{max_words} words.",
        messages=[{"role": "user", "content": f"Summarize:\n\n{text}"}]
    )
    return response.content[0].text
```

## Hierarchical Summarization (Long Documents)

For documents that exceed the context window, chunk and summarize recursively:

```python
def hierarchical_summarize(text: str, chunk_size: int = 3000) -> str:
    """Summarize very long documents by chunking and then synthesizing."""
    words = text.split()
    chunks = [" ".join(words[i:i+chunk_size]) for i in range(0, len(words), chunk_size)]

    if len(chunks) == 1:
        return summarize(text)

    # Summarize each chunk
    chunk_summaries = [summarize(chunk, style="concise", max_words=150) for chunk in chunks]

    # Synthesize chunk summaries into a final summary
    combined = "\n\n".join(f"Section {i+1}:\n{s}" for i, s in enumerate(chunk_summaries))
    response = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=1024,
        system="Synthesize these section summaries into a coherent, unified summary of the full document.",
        messages=[{"role": "user", "content": combined}]
    )
    return response.content[0].text
```

## Multi-Document Synthesis

```python
def synthesize_documents(docs: list[str], question: str | None = None) -> str:
    """Synthesize insights across multiple documents."""
    doc_summaries = [summarize(doc, style="bullets") for doc in docs]
    combined = "\n\n".join(f"Document {i+1}:\n{s}" for i, s in enumerate(doc_summaries))

    if question:
        prompt = f"Based on these documents, answer: {question}\n\nDocuments:\n{combined}"
    else:
        prompt = f"Synthesize the key themes, agreements, and contradictions across these documents:\n{combined}"

    response = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=1024,
        messages=[{"role": "user", "content": prompt}]
    )
    return response.content[0].text
```

## Meeting/Conversation Summary

```python
def summarize_meeting(transcript: str) -> dict:
    response = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=2048,
        tools=[{
            "name": "meeting_summary",
            "description": "Structured meeting summary",
            "input_schema": {
                "type": "object",
                "properties": {
                    "tldr": {"type": "string", "description": "One-sentence summary"},
                    "key_decisions": {"type": "array", "items": {"type": "string"}},
                    "action_items": {
                        "type": "array",
                        "items": {
                            "type": "object",
                            "properties": {
                                "task": {"type": "string"},
                                "owner": {"type": "string"},
                                "due": {"type": "string"}
                            }
                        }
                    },
                    "open_questions": {"type": "array", "items": {"type": "string"}},
                    "next_meeting": {"type": "string"}
                },
                "required": ["tldr", "key_decisions", "action_items"]
            }
        }],
        tool_choice={"type": "tool", "name": "meeting_summary"},
        messages=[{"role": "user", "content": f"Summarize this meeting:\n\n{transcript}"}]
    )
    for block in response.content:
        if block.type == "tool_use":
            return block.input
    return {}
```

## With Prompt Caching (for repeated queries on same doc)

```python
def summarize_with_cache(document: str, queries: list[str]) -> list[str]:
    """Cache the document and answer multiple summary queries cheaply."""
    answers = []
    for query in queries:
        response = client.messages.create(
            model="claude-sonnet-4-6",
            max_tokens=512,
            messages=[{
                "role": "user",
                "content": [
                    {"type": "text", "text": document, "cache_control": {"type": "ephemeral"}},
                    {"type": "text", "text": f"\n{query}"}
                ]
            }]
        )
        answers.append(response.content[0].text)
    return answers
```

## Links

- [Cookbook: capabilities/summarization/](https://github.com/anthropics/claude-cookbooks/tree/main/capabilities/summarization)
- [Cookbook: misc/pdf_upload_summarization.ipynb](https://github.com/anthropics/claude-cookbooks/blob/main/misc/pdf_upload_summarization.ipynb)
