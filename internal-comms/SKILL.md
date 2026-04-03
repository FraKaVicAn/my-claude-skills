---
name: internal-comms
description: Write internal company communications — 3P updates, leadership status reports, newsletters, incident reports — using your company's preferred format
origin: ComposioHQ/awesome-claude-skills
tools: Read, Write, Edit
---

# Internal Comms Skill

Draft internal communications (team updates, company newsletters, incident reports, FAQ docs) following your organization's specific format and tone guidelines.

## When to Activate

- Writing a 3P update (Progress, Plans, Problems) for your team
- Drafting a weekly or monthly company newsletter
- Creating a leadership status report or project update
- Writing an incident report or post-mortem summary
- Answering common internal questions in FAQ format

## Communication Types

| Type | Trigger Keywords |
|------|-----------------|
| 3P Update | "team update", "weekly update", "3P", "progress plans problems" |
| Company Newsletter | "company newsletter", "all-hands update", "company-wide" |
| Leadership Update | "leadership update", "exec summary", "status report" |
| Incident Report | "incident report", "post-mortem", "outage report" |
| FAQ | "common questions", "internal FAQ", "team FAQ" |
| Project Update | "project update", "milestone update" |

## Workflow

1. **Identify type** — match request to communication type above
2. **Load guidelines** — read the relevant format file from `examples/` if available
3. **Gather content** — ask for or read the raw information to include
4. **Draft** — follow the template structure and tone for the type
5. **Review** — check for clarity, completeness, and appropriate tone

## 3P Update Template

```markdown
## Week of [Date] — [Team Name]

### Progress ✅
- [What was completed this week]

### Plans 📋
- [What's happening next week]

### Problems ⚠️
- [Blockers, risks, or help needed]
```

## Tone Guidelines

- **3P / project updates**: factual, concise, bullet-point-first
- **Newsletters**: conversational, positive, storytelling-forward
- **Incident reports**: objective, blameless, action-oriented
- **Leadership updates**: executive summary first, details below

## Links

- [github.com/ComposioHQ/awesome-claude-skills](https://github.com/ComposioHQ/awesome-claude-skills)
