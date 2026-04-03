---
name: competitive-ads-extractor
description: Research competitor advertising strategies from Facebook Ad Library and LinkedIn — extract messaging patterns, categorize by theme, identify what's working
origin: ComposioHQ/awesome-claude-skills
tools: WebSearch, WebFetch, Read, Write, Bash
---

# Competitive Ads Extractor Skill

Systematically analyze competitor ad campaigns: what problems they highlight, how they position, which copy and creative patterns appear most frequently, and what you can adapt (not copy).

## When to Activate

- Launching a new campaign and wanting competitive context
- Identifying messaging angles that resonate in your market
- Understanding how competitors position for different audience segments
- Building a swipe file of effective ad patterns

## Research Sources

- **Facebook Ad Library**: `https://www.facebook.com/ads/library/?active_status=active&ad_type=all&country=US&q=COMPANY`
- **LinkedIn Ad Library**: Available via LinkedIn Campaign Manager transparency
- **Google Ads Transparency Center**

## Workflow

### 1. Identify Competitors
List 3–5 direct competitors and 2–3 category leaders.

### 2. Extract Ads
For each competitor:
- Visit their Ad Library page
- Note: headline, primary text, CTA, visual description, offer type
- Screenshot or transcribe the ad content

### 3. Categorize by Theme
Group ads by:
- Problem highlighted
- Audience segment targeted
- Offer type (trial, demo, discount, social proof)
- Tone (fear, aspiration, urgency, education)

### 4. Identify Patterns
- Which problems come up repeatedly across competitors?
- What CTAs dominate?
- Are there visual patterns (video vs. static, product screenshots vs. lifestyle)?
- Any messaging gaps your brand could own?

## Output Format

```markdown
## Competitor: Acme Corp
**Ads analyzed:** 12 active ads
**Running since:** Some ads active 90+ days (likely performing)

### Top Themes
1. "Save X hours per week" — 6/12 ads (dominant angle)
2. Social proof ("10,000 teams") — 4/12 ads
3. Fear of missing out ("Before your competitor does") — 2/12 ads

### Recommended Angles to Test
- [Gap]: No competitor addresses onboarding friction — opportunity
- [Adaptation]: "10,000 teams" → "Join 500 teams like yours" (specificity)
```

> Use for inspiration and market understanding, not direct copying.

## Links

- [Facebook Ad Library](https://www.facebook.com/ads/library/)
- [github.com/ComposioHQ/awesome-claude-skills](https://github.com/ComposioHQ/awesome-claude-skills)
