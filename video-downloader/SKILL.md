---
name: video-downloader
description: Download YouTube videos with quality/format selection and audio extraction using yt-dlp — auto-installs dependencies
origin: ComposioHQ/awesome-claude-skills
tools: Bash, Read, Write
---

# Video Downloader Skill

Download YouTube videos at any quality, extract audio as MP3, or save in specific formats using yt-dlp (auto-installed if missing).

## When to Activate

- Downloading a YouTube video for offline viewing or editing
- Extracting audio from a video (podcast, lecture, music)
- Saving a video at a specific resolution (720p, 1080p)
- Batch downloading with quality control

## Usage

```bash
python scripts/download_video.py "https://www.youtube.com/watch?v=VIDEO_ID"

# Options
--quality best|1080p|720p|480p|360p|worst   (default: best)
--format mp4|webm|mkv                         (default: mp4)
--audio                                        # extract MP3 only
--output /path/to/dir                          (default: /mnt/user-data/outputs/)
```

## Examples

```bash
# Best quality MP4
python scripts/download_video.py "https://youtu.be/abc123"

# 720p only
python scripts/download_video.py "https://youtu.be/abc123" --quality 720p

# Audio only (MP3)
python scripts/download_video.py "https://youtu.be/abc123" --audio

# Save to specific folder
python scripts/download_video.py "https://youtu.be/abc123" --output ~/Downloads/
```

## Under the Hood

Uses `yt-dlp` which:
- Auto-installs on first run if missing (`pip install yt-dlp`)
- Fetches video metadata and available formats
- Selects best matching stream for your quality preference
- Merges separate video + audio streams when needed (ffmpeg required for 1080p+)

## Constraints

- Single video downloads only (no playlist support)
- Filename derived from video title automatically
- Higher quality = more processing time and storage
- 1080p+ requires ffmpeg: `brew install ffmpeg` / `apt install ffmpeg`

## Links

- [yt-dlp](https://github.com/yt-dlp/yt-dlp)
- [github.com/ComposioHQ/awesome-claude-skills](https://github.com/ComposioHQ/awesome-claude-skills)
