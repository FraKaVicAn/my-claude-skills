---
name: template-skill
description: Starter template for building new Claude Code skills — copy this skeleton and fill in your workflow
origin: ComposioHQ/awesome-claude-skills
tools: Read, Write, Edit, Bash
---

# Template Skill

A minimal skeleton for creating a new Claude Code skill. Copy this directory, rename it, and replace the placeholder content.

## When to Activate

- Starting a new skill from scratch
- Needing a reference for required SKILL.md structure

## How to Use This Template

1. Copy the directory: `cp -r template-skill my-new-skill`
2. Update the frontmatter: `name`, `description`, `tools`
3. Replace this body with your skill's actual instructions
4. Add `scripts/`, `references/`, or `assets/` as needed

## Frontmatter Fields

| Field | Required | Description |
|-------|----------|-------------|
| `name` | Yes | Unique kebab-case identifier |
| `description` | Yes | One-liner used for skill discovery |
| `origin` | No | Source repo URL |
| `tools` | Yes | Comma-separated list of tools the skill uses |

## Body Structure

- **When to Activate** — conditions that trigger this skill
- **Workflow** — step-by-step instructions for Claude to follow
- **Examples** — sample inputs and expected outputs
- **Links** — reference documentation

## Links

- [github.com/ComposioHQ/awesome-claude-skills](https://github.com/ComposioHQ/awesome-claude-skills)
