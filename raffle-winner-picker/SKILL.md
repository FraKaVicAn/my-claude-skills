---
name: raffle-winner-picker
description: Randomly select winners from CSV, Excel, Google Sheets, or plain lists using cryptographically secure random selection — transparent, timestamped, auditable
origin: ComposioHQ/awesome-claude-skills
tools: Read, Write, Bash
---

# Raffle Winner Picker Skill

Fairly and transparently select raffle or contest winners from any list format. Uses cryptographically secure random selection with full audit trail.

## When to Activate

- Social media giveaway winner selection
- Event raffle or prize drawing
- Randomly assigning resources, tasks, or team slots
- Survey participant selection for incentives
- Contest winner picking with verifiable fairness

## Supported Input Formats

- CSV or Excel file (specify column name for entries)
- Google Sheets URL
- Plain text list (one entry per line)
- Manual paste

## Usage

```
Pick 3 winners from entries.csv (column: "email")
Pick 1 winner from this list: Alice, Bob, Carol, Dave, Eve
Select 2 runners-up from giveaway.xlsx, excluding previous winner "frank@example.com"
```

## Core Features

- **Cryptographically secure**: uses `secrets` module (Python) — no predictable patterns
- **Duplicate prevention**: automatically deduplicates entry list before selection
- **Exclude previous winners**: pass a list of prior winners to exclude
- **Weighted entries**: support multiple entries per person (e.g., 2 entries for referrals)
- **Runners-up**: select backup winners in case primary winners don't respond
- **Timestamped results**: every draw records exact time for verification

## Example Output

```markdown
## Raffle Draw — 2025-04-02 14:32:07 UTC

**Winners (3 selected from 847 entries):**
1. 🥇 sarah.j@example.com
2. 🥈 mike_torres_99
3. 🥉 user4421

**Runners-up (2):**
4. backup1@example.com
5. backup2@example.com

**Verification seed**: a3f9c2... (for third-party audit)
**Entries file hash**: sha256:b1d8e4...
```

## Links

- [github.com/ComposioHQ/awesome-claude-skills](https://github.com/ComposioHQ/awesome-claude-skills)
