---
name: langsmith-fetch
description: Debug LangChain and LangGraph agents by fetching and analyzing execution traces from LangSmith — tool calls, memory ops, errors, and performance metrics
origin: ComposioHQ/awesome-claude-skills
tools: Bash, Read, Write
---

# LangSmith Fetch Skill

Retrieve and analyze LangSmith execution traces to debug LangChain/LangGraph agents: why they failed, what tools they called, where they got stuck.

## When to Activate

- "What went wrong with my agent run?"
- "Show me the recent traces for my project"
- Debugging incorrect tool selection or sequencing
- Analyzing memory operation failures
- Investigating performance bottlenecks in an agent pipeline

## Setup

```bash
pip install langsmith-fetch
export LANGSMITH_API_KEY=your_key
export LANGSMITH_PROJECT=your_project_name
# Verify:
langsmith-fetch health
```

## Workflow 1 — Quick Debug (Recent Activity)

```bash
langsmith-fetch runs --limit 10 --format pretty
```

Reports for each run:
- Success/failure status
- Tools called in order
- Token consumption
- Wall-clock time

## Workflow 2 — Deep Trace Analysis

```bash
langsmith-fetch trace <run_id> --include-metadata
```

Examines:
- Full execution flow step-by-step
- Each tool call with inputs and outputs
- Error messages and stack traces
- Root cause hypothesis + remediation suggestions

## Workflow 3 — Export for Team Sharing

```bash
langsmith-fetch export <run_id> --output ./debug-session/
```

Creates:
```
debug-session/
  traces/
  threads/
  metadata.json
```

## Workflow 4 — Error Pattern Analysis

```bash
langsmith-fetch errors --since 7d --group-by type
```

Categorizes failures by type and frequency across multiple runs.

## Common Failure Patterns

| Symptom | Likely Cause | Debug Step |
|---------|-------------|------------|
| Agent loops | Missing stop condition | Inspect tool outputs for empty/null returns |
| Wrong tool selected | Tool descriptions too similar | Fetch trace, compare tool descriptions |
| Memory failure | State serialization error | Check memory operation logs in trace |
| High latency | Redundant tool calls | Count tool calls per run, look for duplicates |

## Links

- [LangSmith](https://smith.langchain.com)
- [github.com/ComposioHQ/awesome-claude-skills](https://github.com/ComposioHQ/awesome-claude-skills)
