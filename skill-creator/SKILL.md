---
name: skill-creator
description: Build modular Claude skill packages — SKILL.md with YAML metadata, optional scripts/references/assets, packaged and ready to share
origin: ComposioHQ/awesome-claude-skills
tools: Read, Write, Edit, Bash
---

# Skill Creator Skill

Create well-structured, shareable Claude skill packages following the standard format: a `SKILL.md` with YAML frontmatter, plus optional `scripts/`, `references/`, and `assets/` directories.

## When to Activate

- Building a new Claude Code skill from scratch
- Packaging an existing workflow into a reusable skill
- Validating that a skill follows the required format

## Skill Package Structure

```
my-skill/
├── SKILL.md          # required — metadata + instructions
├── scripts/          # optional — helper scripts the skill invokes
├── references/       # optional — docs, examples, lookup tables
└── assets/           # optional — images, templates, static files
```

## SKILL.md Format

```markdown
---
name: my-skill
description: One-line description used for skill discovery — be specific
origin: your-github/repo
tools: Read, Write, Edit, Bash, WebSearch
---

# My Skill

Brief intro explaining what this skill does.

## When to Activate

- Situation 1 that should trigger this skill
- Situation 2

## Workflow

Step-by-step instructions...

## Examples

...
```

## Six-Step Workflow

1. **Understand** — clarify what problem the skill solves and when it activates
2. **Plan** — outline the workflow and what supporting files are needed
3. **Initialize** — create the directory and SKILL.md skeleton
4. **Edit** — fill in instructions, examples, and scripts
5. **Package** — validate frontmatter, test the skill end-to-end
6. **Iterate** — refine based on real usage

## Best Practices

- Keep `SKILL.md` lean — progressive disclosure, not an encyclopedia
- The `description` field drives skill discovery; make it searchable
- List only tools the skill actually uses
- Scripts in `scripts/` should be runnable standalone for testability

## Links

- [github.com/ComposioHQ/awesome-claude-skills](https://github.com/ComposioHQ/awesome-claude-skills)
