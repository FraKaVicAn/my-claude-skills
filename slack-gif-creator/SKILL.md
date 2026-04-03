---
name: slack-gif-creator
description: Create Slack-optimized animated GIFs for messages (<2MB) and custom emoji (<64KB) — animation primitives, validators, and auto-optimization
origin: ComposioHQ/awesome-claude-skills
tools: Bash, Read, Write
---

# Slack GIF Creator Skill

Build animated GIFs sized and optimized for Slack: regular message GIFs (≤2MB) and custom emoji (strict ≤64KB). Includes composable animation primitives and automatic size reduction.

## When to Activate

- Creating a custom Slack emoji from an image or animation
- Animating a logo, icon, or reaction GIF for team use
- Optimizing an existing GIF that's too large for Slack

## Size Constraints

| Use | Max Size | Optimal Dimensions |
|-----|----------|--------------------|
| Message GIF | ~2MB | 480×480px |
| Custom emoji | **64KB strict** | 128×128px |

The 64KB emoji limit is non-negotiable — aggressive optimization required.

## Animation Primitives

Compose these freely:

| Primitive | Effect |
|-----------|--------|
| `shake` | Horizontal tremor |
| `bounce` | Vertical bounce |
| `spin` | Rotation |
| `pulse` | Scale in/out |
| `fade` | Opacity in/out |
| `zoom` | Scale up/down |
| `wiggle` | Rotation oscillation |
| `slide` | Directional movement |
| `flip` | Mirror transform |
| `morph` | Shape interpolation |
| `explode` | Radial expansion |

Animation phases: **anticipation → action → reaction**

## GIF Creation (Python)

```python
from PIL import Image, ImageDraw
import imageio, numpy as np

frames = []
for i in range(20):
    img = Image.new("RGBA", (128, 128), (0, 0, 0, 0))
    draw = ImageDraw.Draw(img)
    # apply animation primitive at step i
    scale = 1 + 0.1 * np.sin(i / 20 * 2 * np.pi)  # pulse
    size = int(60 * scale)
    draw.ellipse([64-size, 64-size, 64+size, 64+size], fill=(255, 100, 50))
    frames.append(np.array(img))

imageio.mimsave("emoji.gif", frames, duration=0.05, loop=0)
```

## Optimization for 64KB Emoji

```bash
# Using gifsicle
gifsicle -O3 --colors 32 --lossy=80 input.gif -o emoji.gif

# Check size
ls -lh emoji.gif

# If still too large: fewer frames, fewer colors, smaller dimensions
gifsicle -O3 --colors 16 --lossy=120 --resize 96x96 input.gif -o emoji.gif
```

**Optimization checklist:**
- Max 10–15 frames for emoji
- Max 32–48 colors (avoid gradients)
- Enable duplicate frame removal
- Reduce to 96×96 if needed

## Links

- [gifsicle](https://www.lcdf.org/gifsicle/)
- [github.com/ComposioHQ/awesome-claude-skills](https://github.com/ComposioHQ/awesome-claude-skills)
