---
name: file-organizer
description: Analyze, deduplicate, and reorganize directories — suggests hierarchy, standardizes names, and cleans clutter with user confirmation before any moves
origin: ComposioHQ/awesome-claude-skills
tools: Read, Write, Bash, Glob
---

# File Organizer Skill

Transform messy directories into clean, logical structures. Analyzes current state, detects duplicates, proposes a new hierarchy, and executes only after user confirmation.

## When to Activate

- Downloads folder is overflowing and needs sorting
- Project files are scattered across multiple locations
- Setting up a new workspace or archive structure
- Cleaning up before a backup or handoff

## Seven-Step Process

1. **Scan** — list all files recursively, note types, sizes, and modification dates
2. **Analyze** — detect duplicates (by name + size), identify categories, flag orphans
3. **Propose** — suggest a new directory hierarchy with example placements
4. **Confirm** — present plan to user, wait for approval before touching anything
5. **Execute** — move/rename files according to approved plan
6. **Log** — write a `reorganization_log.txt` with every change made (original → new path)
7. **Report** — summarize: files moved, duplicates removed, space freed

## Typical Structures

**By type:**
```
Downloads/
  docs/      # PDF, DOCX, TXT
  images/    # JPG, PNG, SVG
  code/      # ZIP, scripts
  misc/
```

**By project + year:**
```
Projects/
  2024/
    project-alpha/
    project-beta/
  2025/
    project-gamma/
```

## Safety Rules

- Never delete without explicit confirmation
- Always create the log before executing moves
- Preserve original files when in doubt — copy first, then confirm delete
- Never touch files outside the specified scope directory

## Links

- [github.com/ComposioHQ/awesome-claude-skills](https://github.com/ComposioHQ/awesome-claude-skills)
