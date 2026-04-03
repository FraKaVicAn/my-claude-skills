---
name: notebooklm
description: Programmatic access to Google NotebookLM via Python API and CLI — create notebooks, import sources (URLs/PDF/YouTube/Drive), generate audio/video/slides/quizzes/mindmaps, chat, research, and share. Auto-activates for NotebookLM tasks.
origin: teng-lin/notebooklm-py
tools: Read, Write, Edit, Bash
---

# NotebookLM Python API & CLI Skill

Unofficial Python library for full programmatic access to Google NotebookLM — including features not available in the web UI.

> **Note**: Uses undocumented Google APIs — suitable for prototypes and research, not production.

## Setup

```bash
pip install notebooklm-py

# Authenticate (opens browser once)
notebooklm login

# Or via environment variables
export NOTEBOOKLM_COOKIE="..."   # session cookie
export NOTEBOOKLM_EMAIL="..."    # Google account email
```

## Python API

### Client

```python
import asyncio
from notebooklm import NotebookLMClient

async def main():
    async with NotebookLMClient() as client:
        # All API calls go here
        notebooks = await client.notebooks.list()
        print(notebooks)

asyncio.run(main())
```

### Notebooks

```python
async with NotebookLMClient() as client:
    # Create
    nb = await client.notebooks.create(title="My Research")

    # List all
    notebooks = await client.notebooks.list()

    # Rename
    await client.notebooks.rename(nb.id, "New Title")

    # Get AI summary
    summary = await client.notebooks.get_summary(nb.id)

    # Delete
    await client.notebooks.delete(nb.id)
```

### Sources — Import Content

```python
async with NotebookLMClient() as client:
    nb_id = "your-notebook-id"

    # Add URL
    await client.sources.add_url(nb_id, "https://example.com/article")

    # Add YouTube video
    await client.sources.add_youtube(nb_id, "https://youtube.com/watch?v=...")

    # Add text
    await client.sources.add_text(nb_id, "Raw text content...", title="My Notes")

    # Add PDF or file
    await client.sources.add_file(nb_id, "/path/to/document.pdf")

    # Add Google Drive document
    await client.sources.add_drive(nb_id, "drive_doc_id")

    # Bulk import multiple URLs
    urls = ["https://url1.com", "https://url2.com", "https://url3.com"]
    for url in urls:
        await client.sources.add_url(nb_id, url)
```

### Artifacts — Generate Content

```python
async with NotebookLMClient() as client:
    nb_id = "your-notebook-id"

    # Audio overview (podcast)
    audio = await client.artifacts.generate_audio(
        nb_id,
        format=AudioFormat.MP3,
        length=AudioLength.LONG,
    )
    await client.artifacts.download(audio.id, "/output/podcast.mp3")

    # Video
    video = await client.artifacts.generate_video(
        nb_id,
        style=VideoStyle.DOCUMENTARY,
    )

    # Slide deck (downloadable as PPTX)
    slides = await client.artifacts.generate_slide_deck(
        nb_id,
        format=SlideDeckFormat.PPTX,
        length=SlideDeckLength.MEDIUM,
    )
    await client.artifacts.download(slides.id, "/output/slides.pptx")

    # Quiz
    quiz = await client.artifacts.generate_quiz(
        nb_id,
        quantity=QuizQuantity.MEDIUM,
        difficulty=QuizDifficulty.INTERMEDIATE,
    )
    await client.artifacts.export(quiz.id, ExportType.JSON, "/output/quiz.json")

    # Flashcards
    cards = await client.artifacts.generate_flashcards(nb_id)
    await client.artifacts.export(cards.id, ExportType.MARKDOWN, "/output/flashcards.md")

    # Infographic
    infographic = await client.artifacts.generate_infographic(
        nb_id,
        orientation=InfographicOrientation.LANDSCAPE,
        detail=InfographicDetail.HIGH,
    )

    # Mind map (export as JSON)
    mindmap = await client.artifacts.generate_mind_map(nb_id)
    await client.artifacts.export(mindmap.id, ExportType.JSON, "/output/mindmap.json")

    # Study guide / report
    report = await client.artifacts.generate_report(
        nb_id,
        format=ReportFormat.STUDY_GUIDE,
    )

    # Data table
    table = await client.artifacts.generate_data_table(nb_id)
```

### Chat

```python
async with NotebookLMClient() as client:
    nb_id = "your-notebook-id"

    # Ask a question
    result = await client.chat.ask(
        nb_id,
        "What are the key findings?",
        mode=ChatMode.GROUNDED,
        response_length=ChatResponseLength.DETAILED,
    )
    print(result.answer)
    print(result.references)  # source citations

    # Stream response
    async for chunk in client.chat.ask_stream(nb_id, "Summarize this"):
        print(chunk, end="", flush=True)

    # Filter to specific sources
    result = await client.chat.ask(
        nb_id,
        "What does the PDF say?",
        source_ids=["source-id-1"],
    )
```

### Research

```python
async with NotebookLMClient() as client:
    nb_id = "your-notebook-id"

    # Web research — automatically imports sources
    await client.research.web(
        nb_id,
        query="latest advances in quantum computing 2024",
        mode="deep",  # or "fast"
    )

    # Google Drive research
    await client.research.drive(
        nb_id,
        query="Q4 financial reports",
    )
```

### Sharing

```python
async with NotebookLMClient() as client:
    nb_id = "your-notebook-id"

    # Share with specific user
    await client.sharing.add_user(
        nb_id,
        email="colleague@example.com",
        permission=SharePermission.EDITOR,
    )

    # Enable public sharing
    await client.sharing.set_public(nb_id, access=ShareAccess.VIEWER)

    # Get share status
    status = await client.sharing.get_status(nb_id)
    print(status.shared_users)
```

## CLI Reference

```bash
# Session
notebooklm login                    # authenticate
notebooklm status                   # check auth status

# Notebooks
notebooklm notebook create "My Research"
notebooklm notebook list
notebooklm notebook use <id>        # set active notebook
notebooklm notebook rename <id> "New Name"
notebooklm notebook summary         # get AI summary

# Sources
notebooklm source add-url https://example.com/article
notebooklm source add-youtube https://youtube.com/watch?v=...
notebooklm source add-file ./document.pdf
notebooklm source add-drive <drive_doc_id>
notebooklm source list
notebooklm source research "quantum computing breakthroughs" --mode deep

# Generate content
notebooklm generate audio            # podcast/audio overview
notebooklm generate video            # video
notebooklm generate slides           # slide deck
notebooklm generate quiz             # quiz
notebooklm generate flashcards       # flashcards
notebooklm generate report           # study guide / report
notebooklm generate infographic      # infographic
notebooklm generate mindmap          # mind map
notebooklm generate datatable        # data table

# Artifacts
notebooklm artifact list
notebooklm artifact download <id> ./output.mp3
notebooklm artifact export <id> --format json ./output.json
notebooklm artifact delete <id>

# Chat
notebooklm chat ask "What are the main conclusions?"
notebooklm chat history

# Notes
notebooklm note create "My annotation"
notebooklm note list

# Sharing
notebooklm share add user@example.com --permission editor
notebooklm share status
notebooklm share public --access viewer
```

## Bulk Import Example

```python
import asyncio
from notebooklm import NotebookLMClient

urls = [
    "https://arxiv.org/abs/2401.00001",
    "https://arxiv.org/abs/2401.00002",
    "https://youtube.com/watch?v=dQw4w9WgXcQ",
    "https://example.com/paper.pdf",
]

async def bulk_import(notebook_title: str, sources: list[str]):
    async with NotebookLMClient() as client:
        nb = await client.notebooks.create(title=notebook_title)
        print(f"Created notebook: {nb.id}")

        for url in sources:
            try:
                if "youtube.com" in url or "youtu.be" in url:
                    await client.sources.add_youtube(nb.id, url)
                else:
                    await client.sources.add_url(nb.id, url)
                print(f"✓ Added: {url}")
            except Exception as e:
                print(f"✗ Failed: {url} — {e}")

        # Generate audio overview
        audio = await client.artifacts.generate_audio(nb.id)
        await client.artifacts.download(audio.id, f"./{notebook_title}.mp3")
        print(f"✓ Audio saved")

asyncio.run(bulk_import("Research Bundle", urls))
```

## Sources
- teng-lin/notebooklm-py
- docs/python-api.md
- docs/cli-reference.md
