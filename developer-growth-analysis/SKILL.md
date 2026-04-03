---
name: developer-growth-analysis
description: Analyze your Claude Code chat history to identify coding patterns, technical gaps, and improvement areas — delivers a personalized report to Slack DMs via HackerNews-curated resources
origin: ComposioHQ/awesome-claude-skills
tools: Read, Bash, WebFetch, WebSearch
---

# Developer Growth Analysis Skill

Read your recent Claude Code chat history, identify development patterns and technical gaps, find targeted learning resources from HackerNews, and compile a personalized growth report.

## When to Activate

- Weekly personal retrospective on development work
- Identifying recurring struggles or knowledge gaps
- Getting data-driven feedback without waiting for code reviews
- Tracking technical improvement across projects over time

## Six-Step Analysis

### 1. Read Chat History
```bash
cat ~/.claude/history.jsonl | tail -n 5000
```
Covers the past 24–48 hours of Claude Code sessions.

### 2. Identify Development Patterns
Extract:
- Projects and technologies worked on
- Types of problems solved
- Recurring questions or errors
- Tools and frameworks used

### 3. Detect Improvement Areas
Flag:
- Errors that appeared more than once
- Topics where multiple follow-up questions were needed
- Frameworks or APIs that caused confusion
- Architectural decisions that were reversed or reworked

### 4. Generate Report

```markdown
## Developer Growth Report — [date]

### Work Summary
- Projects: [list]
- Technologies: [list]
- Problem types: debugging 40%, feature dev 35%, architecture 25%

### Improvement Areas (prioritized)
1. **[Topic]** — Evidence: [specific examples from history]
   Recommendation: [actionable next step]

### Strengths Observed
- [Pattern that went smoothly]

### Action Items
- [ ] Read: [specific resource]
- [ ] Practice: [specific exercise]
```

### 5. Find Curated Resources
Search HackerNews for articles matching identified improvement areas.

### 6. Deliver to Slack (optional)
If Composio is configured, send the report as a Slack DM for easy reference.

## Links

- [github.com/ComposioHQ/awesome-claude-skills](https://github.com/ComposioHQ/awesome-claude-skills)
