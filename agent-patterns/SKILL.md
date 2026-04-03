---
name: agent-patterns
description: Implement Anthropic's production agent architectures — orchestrator/workers, evaluator/optimizer, and basic sequential workflows with Claude
origin: anthropics/claude-cookbooks/patterns/agents
tools: Read, Write, Edit, Bash
---

# Agent Patterns Skill

Build robust multi-agent systems using Anthropic's battle-tested architectural patterns: orchestrator/workers for parallelism, evaluator/optimizer for self-improvement, and basic sequential workflows.

## When to Activate

- Task too complex for a single Claude call
- Work can be parallelized across specialized subagents
- Output quality needs iterative refinement
- Building a pipeline where steps depend on prior results

## Pattern 1: Orchestrator / Workers

The orchestrator analyzes the task, decomposes it into subtasks, dispatches to specialized workers, and synthesizes results.

```python
import anthropic
import concurrent.futures

client = anthropic.Anthropic()

def orchestrator(task: str) -> str:
    """Break the task into subtasks."""
    response = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=2048,
        system="""You are an orchestrator. Analyze the task and output subtasks in XML:
<subtasks>
  <subtask><id>1</id><description>...</description><specialist>researcher|writer|reviewer</specialist></subtask>
</subtasks>""",
        messages=[{"role": "user", "content": f"Task: {task}"}]
    )
    return response.content[0].text

def worker(subtask: str, specialist: str) -> str:
    """Execute a single subtask."""
    system_prompts = {
        "researcher": "You are a thorough researcher. Find key facts and data.",
        "writer": "You are a skilled writer. Produce clear, engaging content.",
        "reviewer": "You are a critical reviewer. Identify gaps and improvements."
    }
    response = client.messages.create(
        model="claude-haiku-4-5",   # faster/cheaper for workers
        max_tokens=1024,
        system=system_prompts.get(specialist, "Complete this subtask."),
        messages=[{"role": "user", "content": subtask}]
    )
    return response.content[0].text

def synthesizer(task: str, results: list[str]) -> str:
    """Combine worker outputs into a final answer."""
    combined = "\n\n".join(f"Result {i+1}:\n{r}" for i, r in enumerate(results))
    response = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=2048,
        messages=[{"role": "user", "content": f"Original task: {task}\n\nWorker results:\n{combined}\n\nSynthesize into a final answer."}]
    )
    return response.content[0].text

# Run workers in parallel
subtasks = [("Research X", "researcher"), ("Draft Y", "writer")]
with concurrent.futures.ThreadPoolExecutor() as executor:
    futures = [executor.submit(worker, task, spec) for task, spec in subtasks]
    results = [f.result() for f in futures]
```

## Pattern 2: Evaluator / Optimizer

Generate an initial output, evaluate it against criteria, and iteratively improve until quality threshold is met.

```python
def evaluator_optimizer(task: str, max_iterations: int = 3) -> str:
    # Generate initial output
    draft = client.messages.create(
        model="claude-sonnet-4-6", max_tokens=1024,
        messages=[{"role": "user", "content": task}]
    ).content[0].text

    for i in range(max_iterations):
        # Evaluate
        eval_response = client.messages.create(
            model="claude-sonnet-4-6", max_tokens=512,
            system="Evaluate this output. Score 1-10. Output: <score>N</score><feedback>...</feedback>",
            messages=[{"role": "user", "content": f"Task: {task}\n\nOutput:\n{draft}"}]
        ).content[0].text

        # Parse score
        import re
        score = int(re.search(r"<score>(\d+)</score>", eval_response).group(1))
        if score >= 8:
            break

        feedback = re.search(r"<feedback>(.*?)</feedback>", eval_response, re.DOTALL).group(1)

        # Improve
        draft = client.messages.create(
            model="claude-sonnet-4-6", max_tokens=1024,
            messages=[
                {"role": "user", "content": task},
                {"role": "assistant", "content": draft},
                {"role": "user", "content": f"Improve based on this feedback:\n{feedback}"}
            ]
        ).content[0].text

    return draft
```

## Pattern 3: Basic Sequential Workflow

```python
def sequential_pipeline(input_data: str, steps: list[str]) -> str:
    result = input_data
    for step in steps:
        response = client.messages.create(
            model="claude-haiku-4-5", max_tokens=1024,
            messages=[{"role": "user", "content": f"{step}\n\nInput:\n{result}"}]
        )
        result = response.content[0].text
    return result

# Example: extract → translate → summarize
output = sequential_pipeline(
    raw_text,
    ["Extract the key facts.", "Translate to French.", "Summarize in 3 bullet points."]
)
```

## When to Use Each Pattern

| Pattern | Best For | Avoid When |
|---------|----------|------------|
| Orchestrator/Workers | Parallelizable subtasks, large scope | Simple single-step tasks |
| Evaluator/Optimizer | Quality-critical output, open-ended tasks | Hard latency constraints |
| Sequential | Ordered transformations, clear pipeline | Tasks needing backtracking |

## Links

- [Cookbook: patterns/agents/](https://github.com/anthropics/claude-cookbooks/tree/main/patterns/agents)
- [Building effective agents](https://docs.anthropic.com/en/docs/build-with-claude/agents)
