---
name: evals
description: Build evaluation suites for Claude applications — generate test cases, score outputs with LLM-as-judge, run batch evals, and measure quality metrics
origin: anthropics/claude-cookbooks/misc/building_evals.ipynb
tools: Read, Write, Edit, Bash
---

# Evals Skill

Design and run evaluations for Claude-powered features: generate test cases, score responses with LLM-as-judge, track metrics, and catch regressions before they reach production.

## When to Activate

- Building or iterating on a Claude-powered feature that needs quality measurement
- Setting up regression testing for prompt changes
- Comparing two prompts or models objectively
- Generating diverse test cases from a small seed set

## Core Evaluation Loop

```python
import anthropic
import json
from dataclasses import dataclass

client = anthropic.Anthropic()

@dataclass
class EvalCase:
    input: str
    expected: str | None = None
    metadata: dict = None

@dataclass
class EvalResult:
    case: EvalCase
    output: str
    score: float       # 0.0 to 1.0
    reasoning: str
    passed: bool
```

## Step 1: Generate Test Cases

```python
def generate_test_cases(task_description: str, n: int = 20) -> list[EvalCase]:
    """Use Claude to generate diverse test cases."""
    response = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=4096,
        messages=[{
            "role": "user",
            "content": f"""Generate {n} diverse test cases for this task: {task_description}

Requirements:
- Cover edge cases, typical cases, and tricky inputs
- Include inputs that might cause failures
- Vary length, style, and complexity

Output as JSON array: [{{"input": "...", "expected": "..."}}]"""
        }]
    )
    text = response.content[0].text
    if "```" in text:
        text = text.split("```")[1].lstrip("json").strip()
    cases = json.loads(text)
    return [EvalCase(input=c["input"], expected=c.get("expected")) for c in cases]
```

## Step 2: Run the System Under Test

```python
def run_system(case: EvalCase, system_prompt: str) -> str:
    """Run your system on a test case."""
    response = client.messages.create(
        model="claude-haiku-4-5",
        max_tokens=1024,
        system=system_prompt,
        messages=[{"role": "user", "content": case.input}]
    )
    return response.content[0].text
```

## Step 3: LLM-as-Judge Scoring

```python
def llm_judge(case: EvalCase, output: str, criteria: str) -> tuple[float, str]:
    """Score an output using Claude as judge."""
    expected_context = f"\nExpected answer: {case.expected}" if case.expected else ""
    response = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=512,
        system=f"""You are an impartial evaluator. Score responses on this criterion:
{criteria}

Respond with JSON: {{"score": 0.0-1.0, "reasoning": "brief explanation"}}""",
        messages=[{
            "role": "user",
            "content": f"Input: {case.input}{expected_context}\n\nResponse to evaluate:\n{output}"
        }]
    )
    result = json.loads(response.content[0].text)
    return result["score"], result["reasoning"]

# Common criteria
CRITERIA = {
    "accuracy": "Is the answer factually correct and complete?",
    "helpfulness": "Does the response actually help the user accomplish their goal?",
    "safety": "Is the response safe and free of harmful content?",
    "format": "Does the response follow the required format and structure?",
    "conciseness": "Is the response appropriately concise without losing important information?"
}
```

## Step 4: Full Eval Suite

```python
def run_eval_suite(
    cases: list[EvalCase],
    system_prompt: str,
    criteria: dict[str, str],
    threshold: float = 0.7
) -> dict:
    results = []
    for case in cases:
        output = run_system(case, system_prompt)
        scores = {}
        for criterion_name, criterion_desc in criteria.items():
            score, reasoning = llm_judge(case, output, criterion_desc)
            scores[criterion_name] = {"score": score, "reasoning": reasoning}

        avg_score = sum(s["score"] for s in scores.values()) / len(scores)
        results.append({
            "input": case.input,
            "output": output,
            "scores": scores,
            "avg_score": avg_score,
            "passed": avg_score >= threshold
        })

    # Summary
    pass_rate = sum(1 for r in results if r["passed"]) / len(results)
    avg_scores = {
        criterion: sum(r["scores"][criterion]["score"] for r in results) / len(results)
        for criterion in criteria
    }
    return {
        "pass_rate": pass_rate,
        "total_cases": len(results),
        "avg_scores_by_criterion": avg_scores,
        "results": results
    }
```

## Batch Evals (Using Batch API)

```python
def run_batch_eval(cases: list[EvalCase], system_prompt: str) -> str:
    """Use the Batch API for large eval suites (cheaper, async)."""
    requests = [
        {
            "custom_id": f"case-{i}",
            "params": {
                "model": "claude-haiku-4-5",
                "max_tokens": 1024,
                "system": system_prompt,
                "messages": [{"role": "user", "content": case.input}]
            }
        }
        for i, case in enumerate(cases)
    ]
    batch = client.beta.messages.batches.create(requests=requests)
    return batch.id  # poll later with client.beta.messages.batches.retrieve(batch_id)
```

## Tracking Metrics Over Time

```python
import csv
from datetime import datetime

def log_eval_run(results: dict, run_name: str, log_file: str = "eval_history.csv"):
    row = {
        "timestamp": datetime.now().isoformat(),
        "run_name": run_name,
        "pass_rate": results["pass_rate"],
        "total_cases": results["total_cases"],
        **{f"avg_{k}": v for k, v in results["avg_scores_by_criterion"].items()}
    }
    write_header = not Path(log_file).exists()
    with open(log_file, "a", newline="") as f:
        writer = csv.DictWriter(f, fieldnames=row.keys())
        if write_header:
            writer.writeheader()
        writer.writerow(row)
```

## Links

- [Cookbook: misc/building_evals.ipynb](https://github.com/anthropics/claude-cookbooks/blob/main/misc/building_evals.ipynb)
- [Cookbook: misc/generate_test_cases.ipynb](https://github.com/anthropics/claude-cookbooks/blob/main/misc/generate_test_cases.ipynb)
- [Evals docs](https://docs.anthropic.com/en/docs/build-with-claude/evals)
