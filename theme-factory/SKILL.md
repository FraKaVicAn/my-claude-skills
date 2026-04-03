---
name: theme-factory
description: Apply professional visual themes to artifacts — 10 pre-set coordinated color+font combinations, or generate a custom theme on request
origin: ComposioHQ/awesome-claude-skills
tools: Read, Write, Edit
---

# Theme Factory Skill

Instantly style presentations, documents, and web artifacts with coordinated color palettes and font pairings. Choose from 10 pre-built themes or describe a custom one.

## When to Activate

- "Apply a professional theme to this artifact"
- "Make this look like a tech startup presentation"
- "I want a dark, minimal theme"
- Styling a Claude.ai artifact, HTML page, or markdown document

## 10 Pre-Built Themes

| # | Name | Palette | Mood |
|---|------|---------|------|
| 1 | Ocean Depths | Navy, teal, seafoam | Calm, professional |
| 2 | Sunset Boulevard | Coral, amber, warm cream | Warm, creative |
| 3 | Forest Canopy | Deep green, sage, birch | Natural, grounded |
| 4 | Modern Minimalist | Near-black, white, single accent | Clean, focused |
| 5 | Golden Hour | Gold, warm brown, soft yellow | Premium, optimistic |
| 6 | Arctic Frost | Ice blue, white, silver | Crisp, technical |
| 7 | Desert Rose | Dusty pink, terracotta, sand | Warm, distinctive |
| 8 | Tech Innovation | Electric blue, dark gray, cyan | Modern, energetic |
| 9 | Botanical Garden | Sage, moss, parchment | Organic, refined |
| 10 | Midnight Galaxy | Deep purple, black, starlight silver | Bold, premium |

## Workflow

1. Show theme list with brief descriptions
2. User selects a number or name (or requests custom)
3. Confirm selection
4. Apply: set CSS variables, font imports, component colors

## Application (CSS)

```css
/* Example: Arctic Frost */
:root {
  --primary:    #4a9edd;
  --background: #f8fbff;
  --surface:    #ffffff;
  --text:       #1a2332;
  --accent:     #c0d8f0;
}
h1, h2, h3 { font-family: 'Inter', sans-serif; color: var(--text); }
body { font-family: 'Source Serif 4', serif; background: var(--background); }
```

## Custom Theme

If no preset fits, describe the desired aesthetic:
> "I want something that feels like a Japanese zen garden — restrained, asymmetric, black ink on white rice paper"

The skill generates a new theme, presents it for approval, then applies it.

## Links

- [github.com/ComposioHQ/awesome-claude-skills](https://github.com/ComposioHQ/awesome-claude-skills)
