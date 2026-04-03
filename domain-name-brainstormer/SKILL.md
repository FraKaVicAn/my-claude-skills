---
name: domain-name-brainstormer
description: Generate creative, available domain names for a product or brand across .com, .io, .dev, .ai, .app TLDs — with availability checking
origin: ComposioHQ/awesome-claude-skills
tools: WebSearch, WebFetch, Read, Write
---

# Domain Name Brainstormer Skill

Generate memorable, brand-aligned domain name candidates and check availability across popular TLDs.

## When to Activate

- Naming a new product, startup, or side project
- Finding alternatives when your first-choice domain is taken
- Exploring creative naming directions (portmanteau, metaphor, abbreviation)

## Workflow

### 1. Gather Context
Ask for:
- What the product/service does (one sentence)
- Target audience
- Tone (professional, playful, technical, minimal)
- Any words to include or avoid
- Preferred TLDs (default: .com, .io, .dev, .ai, .app)

### 2. Generate Name Candidates

Strategies to apply:
- **Direct**: clear description of the product (`sendgrid`, `stripe`)
- **Portmanteau**: blend two words (`instagram` = instant + telegram)
- **Metaphor**: evocative concept (`notion`, `figma`)
- **Abbreviation**: initialism or truncation (`gitlab`, `npm`)
- **Made-up word**: invented, brandable (`xerox`, `kodak`)
- **Action verb**: imperative form (`fetch`, `render`)

Generate 20+ candidates per direction.

### 3. Check Availability
For each shortlisted domain, check via WHOIS or registrar search (Namecheap, Google Domains).

### 4. Final Shortlist
Present top 5–10 with:
- Domain name
- Availability status per TLD
- Why it fits the brand
- Potential trademark conflicts to investigate

## Output Example

```
✅ buildfast.io — available
✅ buildfast.dev — available
❌ buildfast.com — taken (parked)

Why: action verb + adjective, easy to spell, strong tech connotations
```

## Links

- [github.com/ComposioHQ/awesome-claude-skills](https://github.com/ComposioHQ/awesome-claude-skills)
