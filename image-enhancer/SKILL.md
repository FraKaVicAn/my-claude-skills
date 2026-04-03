---
name: image-enhancer
description: Upscale, sharpen, and reduce artifacts in images — automatic backups, batch processing, natural language input like "sharpen this screenshot to 4K"
origin: ComposioHQ/awesome-claude-skills
tools: Bash, Read, Write, Glob
---

# Image Enhancer Skill

Improve image quality through upscaling, sharpening, noise reduction, and artifact removal. Works with natural language instructions and always creates backups before modifying originals.

## When to Activate

- "Sharpen this screenshot for a presentation"
- "Upscale this product image to 4K"
- "Reduce JPEG artifacts in these photos"
- Batch processing a folder of images before publishing

## Natural Language Examples

```
"Upscale logo.png to 4x resolution"
"Sharpen all images in ./assets/"
"Reduce noise in scan.jpg and save as PNG"
"Make this screenshot look crisper at 2x"
```

## Workflow

1. **Backup** — copy originals to `./originals_backup/` before any changes
2. **Analyze** — detect current dimensions, format, and quality issues
3. **Process** — apply requested enhancements
4. **Output** — save to specified format/location (default: same directory, `_enhanced` suffix)
5. **Report** — show before/after dimensions and file sizes

## Common Operations

```bash
# Upscale with Real-ESRGAN (if available)
realesrgan-ncnn-vulkan -i input.jpg -o output.jpg -s 4

# Sharpen with ImageMagick
convert input.png -sharpen 0x1.5 output.png

# Reduce JPEG artifacts
convert input.jpg -despeckle -enhance output.jpg

# Batch resize
mogrify -resize 1920x1080 -path ./resized/ *.jpg
```

## Output Formats

- JPEG (web, smaller files)
- PNG (lossless, transparency)
- WebP (modern web, best compression)
- TIFF (print, archival)

## Use Cases

- Blog/social media images
- Presentation screenshots
- Product photography cleanup
- Scanned document enhancement
- Print preparation

## Links

- [github.com/ComposioHQ/awesome-claude-skills](https://github.com/ComposioHQ/awesome-claude-skills)
