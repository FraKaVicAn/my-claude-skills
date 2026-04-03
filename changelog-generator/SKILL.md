---
name: changelog-generator
description: Transform git commit history into polished, customer-ready release notes — categorized, emoji-formatted, and filtered of internal dev noise
origin: ComposioHQ/awesome-claude-skills
tools: Bash, Read, Write
---

# Changelog Generator Skill

Automatically convert raw git commits into clean, user-friendly changelogs. Categorizes changes, translates developer language, and filters internal-only commits.

## When to Activate

- Preparing release notes for a version tag
- Writing weekly/monthly product update summaries
- Creating app store submission changelogs
- Documenting changes for customers or stakeholders

## Usage

```bash
# Last week's changes
git log --since="1 week ago" --pretty=format:"%h %s %an %ad" --date=short

# Between versions
git log v1.2.0..v1.3.0 --pretty=format:"%h %s"

# Specific date range
git log --after="2025-03-01" --before="2025-04-01" --pretty=format:"%h %s %b"
```

## Output Format

```markdown
## v1.3.0 — April 2025

### ✨ New Features
- **Dashboard redesign** — New analytics view with real-time updates
- **CSV export** — Download your data in one click

### 🔧 Improvements
- Search is now 3x faster on large datasets
- Mobile layout is more responsive on small screens

### 🐛 Bug Fixes
- Fixed login timeout on Safari
- Resolved sync issue when editing shared documents

### 🔒 Security
- Updated authentication token expiry policy
```

## Filtering Rules

**Include**: features, improvements, bug fixes, security updates, breaking changes  
**Exclude**: refactors with no user impact, internal tooling changes, test updates, merge commits, dependency bumps (unless security-related)

## Customization

Provide an existing style guide file (e.g., `CHANGELOG.md` or `docs/release-style.md`) and the generator will match your tone, section names, and formatting conventions.

## Links

- [github.com/ComposioHQ/awesome-claude-skills](https://github.com/ComposioHQ/awesome-claude-skills)
