---
name: multimodal-vision
description: Use Claude's vision capabilities — analyze images, read charts/graphs, transcribe text from images, process PDFs, and combine vision with tool use
origin: anthropics/claude-cookbooks/multimodal
tools: Read, Write, Edit, Bash
---

# Multimodal Vision Skill

Send images, PDFs, and documents to Claude for analysis, transcription, chart reading, and visual reasoning.

## When to Activate

- Analyzing images, screenshots, or diagrams
- Extracting text or data from scanned documents
- Reading charts, graphs, or PowerPoint slides
- Processing PDFs with visual content
- Combining image input with tool calls

## Image Input: Base64

```python
import anthropic
import base64

client = anthropic.Anthropic()

def analyze_image(image_path: str, question: str) -> str:
    with open(image_path, "rb") as f:
        image_data = base64.b64encode(f.read()).decode()

    ext = image_path.rsplit(".", 1)[-1].lower()
    media_types = {"jpg": "image/jpeg", "jpeg": "image/jpeg",
                   "png": "image/png", "gif": "image/gif", "webp": "image/webp"}

    response = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=1024,
        messages=[{
            "role": "user",
            "content": [
                {
                    "type": "image",
                    "source": {
                        "type": "base64",
                        "media_type": media_types[ext],
                        "data": image_data
                    }
                },
                {"type": "text", "text": question}
            ]
        }]
    )
    return response.content[0].text
```

## Image Input: URL

```python
response = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=512,
    messages=[{
        "role": "user",
        "content": [
            {
                "type": "image",
                "source": {"type": "url", "url": "https://example.com/chart.png"}
            },
            {"type": "text", "text": "What trend does this chart show? Extract the key data points."}
        ]
    }]
)
```

## Multiple Images

```python
def compare_images(paths: list[str], question: str) -> str:
    content = []
    for path in paths:
        with open(path, "rb") as f:
            data = base64.b64encode(f.read()).decode()
        content.append({
            "type": "image",
            "source": {"type": "base64", "media_type": "image/jpeg", "data": data}
        })
    content.append({"type": "text", "text": question})

    response = client.messages.create(
        model="claude-sonnet-4-6", max_tokens=1024,
        messages=[{"role": "user", "content": content}]
    )
    return response.content[0].text
```

## PDF Processing

```python
import httpx

# From URL
pdf_url = "https://example.com/report.pdf"
pdf_data = base64.b64encode(httpx.get(pdf_url).content).decode()

response = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=2048,
    messages=[{
        "role": "user",
        "content": [
            {
                "type": "document",
                "source": {"type": "base64", "media_type": "application/pdf", "data": pdf_data}
            },
            {"type": "text", "text": "Summarize the key findings and extract any tables."}
        ]
    }]
)
```

## Text Transcription from Images

```python
def transcribe_image(image_path: str) -> str:
    return analyze_image(
        image_path,
        "Transcribe all text visible in this image exactly as it appears. "
        "Preserve formatting, line breaks, and structure."
    )
```

## Chart / Graph Reading

```python
def read_chart(chart_path: str) -> dict:
    response = analyze_image(
        chart_path,
        """Analyze this chart and provide:
1. Chart type
2. Title and axis labels
3. Key data points and values (extract all numbers you can see)
4. Main trend or insight
5. Any notable anomalies

Format as structured data."""
    )
    return response
```

## Vision + Tool Use

```python
tools = [{
    "name": "save_extracted_data",
    "description": "Save structured data extracted from an image",
    "input_schema": {
        "type": "object",
        "properties": {
            "data": {"type": "object"},
            "source_image": {"type": "string"}
        },
        "required": ["data"]
    }
}]

response = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    tools=tools,
    messages=[{
        "role": "user",
        "content": [
            {"type": "image", "source": {"type": "base64", "media_type": "image/png", "data": img_data}},
            {"type": "text", "text": "Extract all data from this invoice and save it."}
        ]
    }]
)
```

## Best Practices

- Images up to 5MB per image, up to 20 images per request
- For high-detail analysis, use full resolution (don't compress unnecessarily)
- For text extraction, ensure image resolution is high enough to read clearly
- The crop tool in `multimodal/crop_tool.ipynb` helps focus on specific regions

## Links

- [Cookbook: multimodal/](https://github.com/anthropics/claude-cookbooks/tree/main/multimodal)
- [Vision docs](https://docs.anthropic.com/en/docs/build-with-claude/vision)
