---
name: connect-apps
description: Integrate Claude Code with 1000+ external apps (Gmail, Slack, GitHub, Notion, Jira, etc.) via Composio — take real actions, not just suggestions
origin: ComposioHQ/awesome-claude-skills
tools: Bash, Read, Write
---

# Connect Apps Skill

Enable Claude Code to take real actions across Gmail, Slack, GitHub, Notion, Jira, and 1000+ services using Composio's tool router. OAuth is handled automatically.

## When to Activate

- Sending emails, Slack messages, or calendar invites from within Claude Code
- Creating GitHub issues, PRs, or Notion pages as part of a workflow
- Updating databases, spreadsheets, or CRM records
- Automating multi-step cross-app workflows from a single prompt

## Setup

```bash
# Install Composio
pip install composio-core

# Authenticate
export COMPOSIO_API_KEY=your_key_from_platform.composio.dev
```

## Usage

```python
from composio_claude import ComposioToolSet, App

toolset = ComposioToolSet()
tools = toolset.get_tools(apps=[App.GMAIL, App.SLACK, App.GITHUB])

# Pass tools to your Claude agent
```

First use of any app triggers a one-time OAuth authorization link. Connections persist after that.

## Supported App Categories

- **Email**: Gmail, Outlook
- **Chat**: Slack, Discord, Teams
- **Dev**: GitHub, GitLab, Linear, Jira
- **Docs**: Notion, Google Docs, Confluence
- **Databases**: Airtable, Supabase, PostgreSQL
- **CRM**: Salesforce, HubSpot
- **Social**: Twitter/X, LinkedIn

## Links

- [platform.composio.dev](https://platform.composio.dev) — get API key
- [Composio docs](https://docs.composio.dev)
- [awesome-claude-skills](https://github.com/ComposioHQ/awesome-claude-skills)
