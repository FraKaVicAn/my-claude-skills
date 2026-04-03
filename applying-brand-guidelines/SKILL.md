---
name: applying-brand-guidelines
description: This skill applies consistent corporate branding and styling to all generated documents — colors, fonts, layouts, and messaging standards for PowerPoint, Excel, and PDF
origin: anthropics/claude-cookbooks/skills/custom_skills/applying-brand-guidelines
tools: Read, Write, Edit, Bash
---

# Corporate Brand Guidelines Skill

Ensures all generated documents adhere to corporate brand standards for consistent, professional communication. Configured here for **Acme Corporation** — adapt color/font values for your own brand.

## When to Activate

Activates when creating or formatting: PowerPoint presentations, Excel spreadsheets, PDF reports, Word documents, or any output requiring consistent corporate styling.

## Brand Identity (Acme Corporation Template)

**Tagline**: "Innovation Through Excellence" | **Industry**: Technology Solutions

## Color Palette

```python
BRAND_COLORS = {
    # Primary
    "acme_blue":   "#0066CC",   # Headers, primary buttons
    "acme_navy":   "#003366",   # Text, accents
    "white":       "#FFFFFF",   # Backgrounds, reverse text
    # Secondary
    "success":     "#28A745",   # Positive metrics
    "warning":     "#FFC107",   # Cautions
    "error":       "#DC3545",   # Negative values
    "neutral_gray":"#6C757D",   # Secondary text
    "row_alt":     "#F8F9FA",   # Alternating table rows
}
```

## Typography

| Element | Size | Weight | Color |
|---------|------|--------|-------|
| H1 | 32pt | Bold | Acme Blue |
| H2 | 24pt | Semibold | Acme Navy |
| H3 | 18pt | Semibold | Acme Navy |
| Body | 11pt | Regular | Acme Navy |
| Caption | 9pt | Regular | Neutral Gray |

**Font stack**: Segoe UI, system-ui, -apple-system, sans-serif

## Document Standards

### PowerPoint
- **Title Slide**: Logo top-left, title, date, presenter name
- **Section Divider**: Blue background, white text
- **Content Slide**: Blue title bar, white content area, max 6 bullets
- **Data Slide**: Charts use brand colors only, no 3D effects

### Excel
- **Row 1 headers**: Bold, white text on Acme Blue (`#0066CC`)
- **Subheaders**: Bold, Acme Navy text
- **Data cells**: Regular, Acme Navy
- **Borders**: Thin, Neutral Gray
- **Alternating rows**: `#F8F9FA`
- **Charts**: Primary = Acme Blue, Secondary = Success Green, no gradients

### PDF / Word
- **Header**: Logo left | Title center | Page number right
- **Footer**: Copyright left | Date center | Classification right
- **Margins**: 1 inch all sides | Line spacing: 1.15 | Para spacing: 12pt after

## Content Guidelines

### Tone
- **Professional** — formal but approachable
- **Clear** — avoid jargon, prefer simple language
- **Active voice** — action-oriented phrasing
- **Positive** — focus on solutions and benefits

### Number Formatting
- Thousands separator: `$1,234,567`
- Currency: `$X,XXX.XX`
- Percentages: `XX.X%` (one decimal)
- Dates: `Month DD, YYYY`

## Logo Rules
- Position: Top-left, first page/slide only
- Width: 120px (maintain aspect ratio)
- Clear space: 20px padding all sides
- Never distort, rotate, recolor, or apply effects

## Prohibited Elements
- Clip art or unapproved stock photos
- Comic Sans, Papyrus, or decorative fonts
- Rainbow colors, gradients, or drop shadows
- Competitor branding or references

## Quality Checklist

Before finalizing any document:
- [ ] Logo properly placed and sized
- [ ] All colors match brand palette exactly
- [ ] Fonts consistent throughout (check headers, body, captions)
- [ ] No typos or grammatical errors
- [ ] Numbers formatted correctly (commas, decimal places)
- [ ] Professional tone maintained
- [ ] Tables have brand-colored headers and alternating rows
- [ ] Charts use only brand colors

## Customizing for Your Brand

Replace the values in this skill with your organization's brand:

```markdown
# Replace these values:
Primary color: #0066CC → your brand primary
Secondary color: #003366 → your brand secondary
Font: Segoe UI → your brand font
Logo path: logo.png → your logo file
Company name: Acme Corporation → your company name
Tagline: "Innovation Through Excellence" → your tagline
```

## Example Prompts

- "Format this Excel report using our brand guidelines"
- "Create a PowerPoint presentation following Acme brand standards"
- "Check this document for brand compliance"
- "Apply brand colors and fonts to this report"

## Links

- [Cookbook: skills/custom_skills/applying-brand-guidelines](https://github.com/anthropics/claude-cookbooks/tree/main/skills/custom_skills/applying-brand-guidelines)
