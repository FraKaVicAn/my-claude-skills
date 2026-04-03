---
name: structured-json-output
description: Extract structured JSON from Claude using tool use schemas, JSON mode, or Pydantic models — reliable, typed output for any data extraction task
origin: anthropics/claude-cookbooks/tool_use/extracting_structured_json.ipynb
tools: Read, Write, Edit, Bash
---

# Structured JSON Output Skill

Get reliable, typed JSON output from Claude using tool use as a schema enforcer, JSON mode, or Pydantic validation.

## When to Activate

- Extracting structured data from unstructured text (articles, emails, documents)
- Building pipelines that need machine-readable output
- Replacing fragile regex parsing with schema-validated extraction
- Any task where you need guaranteed field types

## Method 1: Tool Use as Schema (Recommended)

Define a tool whose input schema describes the structure you want. Claude fills it in.

```python
import anthropic
import json

client = anthropic.Anthropic()

def extract_structured(text: str, schema: dict, description: str) -> dict:
    """Extract structured data using tool use as a schema enforcer."""
    response = client.messages.create(
        model="claude-haiku-4-5",   # fast and cheap for extraction
        max_tokens=1024,
        tools=[{
            "name": "extract_data",
            "description": description,
            "input_schema": schema
        }],
        tool_choice={"type": "tool", "name": "extract_data"},  # force tool use
        messages=[{"role": "user", "content": f"Extract from this text:\n\n{text}"}]
    )

    for block in response.content:
        if block.type == "tool_use":
            return block.input
    return {}

# Example: Extract article metadata
article_schema = {
    "type": "object",
    "properties": {
        "title": {"type": "string"},
        "author": {"type": "string"},
        "topics": {"type": "array", "items": {"type": "string"}},
        "summary": {"type": "string", "description": "2-3 sentence summary"},
        "sentiment": {"type": "string", "enum": ["positive", "negative", "neutral"]},
        "word_count_estimate": {"type": "integer"}
    },
    "required": ["title", "topics", "summary", "sentiment"]
}

result = extract_structured(article_text, article_schema, "Extract metadata from the article")
print(json.dumps(result, indent=2))
```

## Method 2: JSON Mode (Prompt-Based)

```python
def json_mode(prompt: str, schema_description: str) -> dict:
    response = client.messages.create(
        model="claude-haiku-4-5",
        max_tokens=1024,
        system=f"You are a data extraction assistant. Always respond with valid JSON matching this structure:\n{schema_description}\n\nOutput only the JSON object, no explanation.",
        messages=[{"role": "user", "content": prompt}]
    )
    text = response.content[0].text.strip()
    # Strip markdown code fences if present
    if text.startswith("```"):
        text = text.split("\n", 1)[1].rsplit("```", 1)[0].strip()
    return json.loads(text)
```

## Method 3: Pydantic Validation

```python
from pydantic import BaseModel, Field
from typing import Optional

class InvoiceData(BaseModel):
    vendor: str
    invoice_number: str
    date: str = Field(description="ISO format YYYY-MM-DD")
    total_amount: float
    currency: str = "USD"
    line_items: list[dict] = Field(default_factory=list)
    tax_amount: Optional[float] = None

def extract_invoice(invoice_text: str) -> InvoiceData:
    schema = InvoiceData.model_json_schema()
    raw = extract_structured(
        invoice_text,
        schema,
        "Extract invoice data including all line items and totals"
    )
    return InvoiceData(**raw)

invoice = extract_invoice(invoice_text)
print(f"Vendor: {invoice.vendor}, Total: {invoice.total_amount} {invoice.currency}")
```

## Batch Extraction

```python
def batch_extract(texts: list[str], schema: dict, description: str) -> list[dict]:
    """Extract from multiple texts efficiently."""
    results = []
    for text in texts:
        try:
            result = extract_structured(text, schema, description)
            results.append({"success": True, "data": result})
        except Exception as e:
            results.append({"success": False, "error": str(e), "text_preview": text[:100]})
    return results
```

## Common Schemas

```python
# Contact extraction
contact_schema = {
    "type": "object",
    "properties": {
        "name": {"type": "string"},
        "email": {"type": "string"},
        "phone": {"type": "string"},
        "company": {"type": "string"},
        "role": {"type": "string"}
    }
}

# Sentiment + topics
sentiment_schema = {
    "type": "object",
    "properties": {
        "sentiment": {"type": "string", "enum": ["positive", "negative", "neutral", "mixed"]},
        "confidence": {"type": "number", "minimum": 0, "maximum": 1},
        "topics": {"type": "array", "items": {"type": "string"}},
        "key_phrases": {"type": "array", "items": {"type": "string"}}
    }
}

# Event extraction
event_schema = {
    "type": "object",
    "properties": {
        "events": {
            "type": "array",
            "items": {
                "type": "object",
                "properties": {
                    "name": {"type": "string"},
                    "date": {"type": "string"},
                    "location": {"type": "string"},
                    "participants": {"type": "array", "items": {"type": "string"}}
                }
            }
        }
    }
}
```

## Links

- [Cookbook: tool_use/extracting_structured_json.ipynb](https://github.com/anthropics/claude-cookbooks/blob/main/tool_use/extracting_structured_json.ipynb)
- [Tool use docs](https://docs.anthropic.com/en/docs/build-with-claude/tool-use)
