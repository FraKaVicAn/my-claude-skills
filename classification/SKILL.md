---
name: classification
description: Classify text into categories with Claude — zero-shot, few-shot, multi-label, and confidence-scored classification with structured output
origin: anthropics/claude-cookbooks/capabilities/classification
tools: Read, Write, Edit, Bash
---

# Classification Skill

Classify text, documents, or data into predefined categories using Claude. Supports zero-shot, few-shot, multi-label, and hierarchical classification with confidence scores.

## When to Activate

- Routing support tickets to the right team
- Categorizing emails, reviews, or feedback
- Content moderation and policy classification
- Intent detection for chatbots
- Tagging documents for search or workflow

## Zero-Shot Classification

```python
import anthropic
import json

client = anthropic.Anthropic()

def classify(text: str, categories: list[str], description: str = "") -> dict:
    """Classify text into one of the given categories."""
    categories_list = "\n".join(f"- {c}" for c in categories)
    response = client.messages.create(
        model="claude-haiku-4-5",   # fast and cheap for classification
        max_tokens=256,
        tools=[{
            "name": "classify",
            "description": "Classify the input text",
            "input_schema": {
                "type": "object",
                "properties": {
                    "category": {"type": "string", "enum": categories},
                    "confidence": {"type": "number", "minimum": 0, "maximum": 1},
                    "reasoning": {"type": "string"}
                },
                "required": ["category", "confidence"]
            }
        }],
        tool_choice={"type": "tool", "name": "classify"},
        system=f"Classify the text into exactly one category.{f' Context: {description}' if description else ''}\n\nCategories:\n{categories_list}",
        messages=[{"role": "user", "content": text}]
    )
    for block in response.content:
        if block.type == "tool_use":
            return block.input
    return {}
```

## Multi-Label Classification

```python
def classify_multi_label(text: str, labels: list[str], threshold: float = 0.5) -> list[str]:
    """Assign multiple labels to text."""
    response = client.messages.create(
        model="claude-haiku-4-5",
        max_tokens=512,
        tools=[{
            "name": "classify_multi",
            "description": "Assign relevant labels to the text",
            "input_schema": {
                "type": "object",
                "properties": {
                    "labels": {
                        "type": "array",
                        "items": {
                            "type": "object",
                            "properties": {
                                "label": {"type": "string", "enum": labels},
                                "score": {"type": "number", "minimum": 0, "maximum": 1}
                            }
                        }
                    }
                },
                "required": ["labels"]
            }
        }],
        tool_choice={"type": "tool", "name": "classify_multi"},
        messages=[{"role": "user", "content": f"Assign all relevant labels:\n\n{text}"}]
    )
    for block in response.content:
        if block.type == "tool_use":
            return [l["label"] for l in block.input["labels"] if l["score"] >= threshold]
    return []
```

## Few-Shot Classification

```python
def classify_few_shot(text: str, examples: list[dict], categories: list[str]) -> dict:
    """Classify with examples for better accuracy."""
    examples_text = "\n".join(
        f'Text: "{ex["text"]}"\nCategory: {ex["label"]}' for ex in examples
    )
    return classify(
        text=f"Examples:\n{examples_text}\n\nClassify this:\n{text}",
        categories=categories
    )
```

## Batch Classification

```python
def batch_classify(texts: list[str], categories: list[str]) -> list[dict]:
    """Classify multiple texts, handling failures gracefully."""
    results = []
    for text in texts:
        try:
            result = classify(text, categories)
            results.append({"text": text[:100], "success": True, **result})
        except Exception as e:
            results.append({"text": text[:100], "success": False, "error": str(e)})
    return results

def aggregate_results(results: list[dict]) -> dict:
    """Summarize classification results."""
    from collections import Counter
    successful = [r for r in results if r.get("success")]
    category_counts = Counter(r["category"] for r in successful)
    avg_confidence = sum(r.get("confidence", 0) for r in successful) / len(successful) if successful else 0
    return {
        "total": len(results),
        "successful": len(successful),
        "distribution": dict(category_counts),
        "avg_confidence": round(avg_confidence, 3)
    }
```

## Common Classification Schemas

```python
# Support ticket routing
SUPPORT_CATEGORIES = ["billing", "technical", "account", "feature_request", "general"]

# Sentiment
SENTIMENT = ["positive", "negative", "neutral", "mixed"]

# Email intent
EMAIL_INTENTS = ["question", "complaint", "praise", "unsubscribe", "spam", "other"]

# Content moderation
MODERATION = ["safe", "adult", "violence", "hate_speech", "spam"]
```

## Links

- [Cookbook: capabilities/classification/](https://github.com/anthropics/claude-cookbooks/tree/main/capabilities/classification)
- [Anthropic API docs](https://docs.anthropic.com)
