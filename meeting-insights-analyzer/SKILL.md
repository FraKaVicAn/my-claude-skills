---
name: meeting-insights-analyzer
description: Analyze meeting transcripts to reveal behavioral patterns — conflict avoidance, speaking ratios, filler words, listening effectiveness, and leadership style
origin: ComposioHQ/awesome-claude-skills
tools: Read, Write, Glob
---

# Meeting Insights Analyzer Skill

Surface behavioral patterns in meeting transcripts: what you avoid saying, how much you talk vs. listen, your facilitation style, and how to improve.

## When to Activate

- Post-meeting reflection on communication patterns
- Coaching yourself or a team member on facilitation
- Performance review preparation
- Tracking communication improvement over time
- Identifying recurring conflict avoidance behaviors

## Supported Transcript Formats

`.txt`, `.md`, `.vtt` (WebVTT), `.srt`, `.docx`

## Analysis Categories

### Conflict Avoidance Detection
Signs: hedging language ("maybe", "kind of", "it might be worth"), indirect framing, topics dropped without resolution, passive voice on difficult subjects.

### Speaking Ratios
- Who talked how much (% of total words)
- Interruption count per participant
- Average speaking turn length

### Filler Word Frequency
Tracks: "um", "uh", "like", "you know", "basically", "literally", "honestly", "actually".

### Active Listening Indicators
Signs of good listening: follow-up questions, paraphrasing, referencing prior speaker's point.
Signs of poor listening: pivoting without acknowledgment, repetitive points.

### Leadership & Facilitation
- Did the meeting have a clear agenda?
- Were decisions captured?
- Were action items assigned with owners?
- Were quieter participants invited to speak?

## Output Format

```markdown
## Meeting Analysis — [date/title]

### Speaking Ratios
| Participant | % of words | Interruptions |
|------------|-----------|---------------|
| Alice | 45% | 3 |
| Bob | 30% | 1 |
| Carol | 25% | 0 |

### Conflict Avoidance — 4 instances
- [12:34] "It might be worth considering..." → Direct alternative: "I recommend..."
- [18:22] Topic dropped after pushback → Could have said: "Let's resolve this before moving on"

### Filler Words
"like" — 23x | "um" — 11x | "you know" — 8x

### Strengths
- Strong follow-up questions in the first half
- Clear action items captured at close

### Top 3 Improvement Actions
1. Replace hedging phrases with direct recommendations
2. Reduce "like" usage — practice pausing instead
3. Invite Carol to speak when she goes quiet for 5+ minutes
```

## Multi-Meeting Trend Tracking

Provide transcripts from multiple meetings to see improvement over time.

## Links

- [github.com/ComposioHQ/awesome-claude-skills](https://github.com/ComposioHQ/awesome-claude-skills)
