---
name: batch-processing
description: Process thousands of Claude requests efficiently using the Batch API — 50% cheaper, async execution, ideal for classification, extraction, and eval pipelines
origin: anthropics/claude-cookbooks/misc/batch_processing.ipynb
tools: Read, Write, Edit, Bash
---

# Batch Processing Skill

Use the Anthropic Batch API to process large volumes of requests asynchronously at 50% reduced cost. Ideal for offline workloads: classification, extraction, eval runs, and data transformation.

## When to Activate

- Processing 100+ items that don't need real-time responses
- Running nightly data pipelines with Claude
- Executing eval suites across large datasets
- Classifying or transforming a large corpus cheaply

## How the Batch API Works

1. Submit up to 10,000 requests in one batch
2. Requests process within 24 hours (usually much faster)
3. Retrieve results when complete
4. Cost: 50% of standard API pricing

## Submit a Batch

```python
import anthropic
import json

client = anthropic.Anthropic()

def create_batch(items: list[dict], system_prompt: str, model: str = "claude-haiku-4-5") -> str:
    """
    items: list of dicts with 'id' and 'text' fields
    Returns batch_id for polling
    """
    requests = [
        {
            "custom_id": item["id"],
            "params": {
                "model": model,
                "max_tokens": 1024,
                "system": system_prompt,
                "messages": [{"role": "user", "content": item["text"]}]
            }
        }
        for item in items
    ]

    batch = client.beta.messages.batches.create(requests=requests)
    print(f"Batch created: {batch.id}")
    print(f"Status: {batch.processing_status}")
    return batch.id
```

## Poll for Completion

```python
import time

def wait_for_batch(batch_id: str, poll_interval: int = 60) -> str:
    """Poll until batch completes. Returns final status."""
    while True:
        batch = client.beta.messages.batches.retrieve(batch_id)
        print(f"Status: {batch.processing_status} | "
              f"Succeeded: {batch.request_counts.succeeded} | "
              f"Errored: {batch.request_counts.errored} | "
              f"Processing: {batch.request_counts.processing}")

        if batch.processing_status == "ended":
            return batch.processing_status

        time.sleep(poll_interval)
```

## Retrieve Results

```python
def get_batch_results(batch_id: str) -> dict[str, str]:
    """Returns dict of custom_id → response text."""
    results = {}
    for result in client.beta.messages.batches.results(batch_id):
        if result.result.type == "succeeded":
            results[result.custom_id] = result.result.message.content[0].text
        else:
            results[result.custom_id] = f"ERROR: {result.result.error.type}"
    return results
```

## Full Pipeline Example

```python
def batch_classify(texts: list[str], categories: list[str]) -> list[dict]:
    """Classify a large list of texts using the Batch API."""
    items = [{"id": f"item-{i}", "text": text} for i, text in enumerate(texts)]

    system = f"""Classify the text into exactly one category.
Categories: {', '.join(categories)}
Respond with JSON only: {{"category": "...", "confidence": 0.0-1.0}}"""

    batch_id = create_batch(items, system)
    wait_for_batch(batch_id)
    raw_results = get_batch_results(batch_id)

    classified = []
    for item in items:
        raw = raw_results.get(item["id"], "")
        try:
            data = json.loads(raw)
            classified.append({"text": item["text"], **data})
        except:
            classified.append({"text": item["text"], "category": "error", "confidence": 0})
    return classified

# Usage
results = batch_classify(my_texts, ["billing", "technical", "general"])
```

## Batch Eval Runner

```python
def run_batch_eval(eval_cases: list[dict], system_prompt: str) -> str:
    """Submit an eval suite as a batch job."""
    items = [{"id": f"eval-{i}", "text": case["input"]} for i, case in enumerate(eval_cases)]
    return create_batch(items, system_prompt, model="claude-haiku-4-5")
```

## Save & Resume

```python
def save_batch_id(batch_id: str, filename: str = ".batch_id"):
    """Save batch ID to resume later."""
    with open(filename, "w") as f:
        f.write(batch_id)

def load_batch_id(filename: str = ".batch_id") -> str | None:
    try:
        return open(filename).read().strip()
    except FileNotFoundError:
        return None

# Resume a batch started in a previous session
batch_id = load_batch_id() or create_batch(items, system)
save_batch_id(batch_id)
wait_for_batch(batch_id)
```

## Best Practices

- Use `claude-haiku-4-5` for batch jobs (fastest + cheapest)
- Group similar tasks in one batch for easier result parsing
- Save `batch_id` to disk — batches persist for 24h even if your script crashes
- Check `request_counts.errored` — retry failed items individually if needed
- Batches with > 1,000 items: monitor `processing` count to estimate ETA

## Links

- [Cookbook: misc/batch_processing.ipynb](https://github.com/anthropics/claude-cookbooks/blob/main/misc/batch_processing.ipynb)
- [Batch API docs](https://docs.anthropic.com/en/docs/build-with-claude/batch-processing)
