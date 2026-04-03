---
name: skill-share
description: Create Claude skills and automatically share them to a Slack channel via Rube — validates format, packages as zip, notifies team
origin: ComposioHQ/awesome-claude-skills
tools: Read, Write, Edit, Bash
---

# Skill Share Skill

Build properly structured Claude skills and automatically distribute them to your team via Slack using Rube integration. Handles creation, validation, packaging, and notification in one workflow.

## When to Activate

- Creating a new skill that should be shared with the team
- Publishing an updated skill version to a Slack channel
- Validating and packaging an existing skill for distribution

## Requirements

- Python 3.7+
- Slack workspace access via Rube
- File system write permissions
- A designated Slack channel for skill announcements

## Workflow

### 1. Initialize
```bash
mkdir my-skill && cd my-skill
# Create SKILL.md with required frontmatter
```

### 2. Validate
The skill checks:
- SKILL.md has valid YAML frontmatter (`name`, `description`, `tools`)
- Naming conventions are followed (kebab-case)
- Required fields are present and non-empty

### 3. Package
```bash
# Creates my-skill.zip with all contents
zip -r my-skill.zip my-skill/
```

### 4. Notify via Slack
Sends a message to the configured channel with:
- Skill name and description
- Author and version
- Discovery instructions for teammates

## Slack Message Format

```
🆕 New skill published: **my-skill**
> One-line description from SKILL.md
Install: /plugin install my-skill
```

## Links

- [github.com/ComposioHQ/awesome-claude-skills](https://github.com/ComposioHQ/awesome-claude-skills)
