---
name: brand-guidelines
description: Apply Anthropic brand styling to artifacts — correct color palette, Poppins/Lora typography, accent colors, and consistent visual identity
origin: ComposioHQ/awesome-claude-skills
tools: Read, Write, Edit
---

# Brand Guidelines Skill

Apply Anthropic's official visual identity to Claude.ai artifacts: correct colors, typefaces, and accent usage for consistent, professional output.

## When to Activate

- Creating artifacts that should match Anthropic's brand
- Reviewing generated UI for brand compliance
- Applying consistent styling across multiple artifacts

## Color Palette

```css
/* Foundational */
--dark:       #141413;
--light:      #faf9f5;
--mid-gray:   #b0aea5;
--light-gray: #e8e6dc;

/* Accents (cycle through these) */
--orange: #d97757;
--blue:   #6a9bcc;
--green:  #788c5d;
```

## Typography

| Element | Font | Fallback | Size |
|---------|------|----------|------|
| Headings (H1–H3) | Poppins | Arial, sans-serif | 24px+ |
| Body text | Lora | Georgia, serif | 16px |
| Code | Monospace | Courier New | 14px |

```css
h1, h2, h3 { font-family: 'Poppins', Arial, sans-serif; }
body, p    { font-family: 'Lora', Georgia, serif; }
```

## Usage Rules

- **Background**: use `--light` (#faf9f5) for main backgrounds, `--dark` for inverse sections
- **Text**: `--dark` on light backgrounds; `--light` on dark backgrounds
- **Accents**: rotate orange → blue → green for visual variety; never use all three in one component
- **Borders / dividers**: `--light-gray` (#e8e6dc)
- **Secondary text**: `--mid-gray` (#b0aea5)

## Sample CSS Reset

```css
:root {
  --dark: #141413; --light: #faf9f5;
  --mid-gray: #b0aea5; --light-gray: #e8e6dc;
  --orange: #d97757; --blue: #6a9bcc; --green: #788c5d;
}
body {
  background: var(--light);
  color: var(--dark);
  font-family: 'Lora', Georgia, serif;
}
h1, h2, h3 {
  font-family: 'Poppins', Arial, sans-serif;
}
```

## Links

- [github.com/ComposioHQ/awesome-claude-skills](https://github.com/ComposioHQ/awesome-claude-skills)
