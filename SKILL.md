---
name: notebooklm
description: >
  Use Google NotebookLM (via the notebooklm-py CLI) to generate content from documents and URLs:
  audio podcasts, videos, quizzes, flashcards, slide decks, infographics, mind maps, reports,
  blog posts, and data tables. Also handles NotebookLM notebook management, sharing, and
  AI-powered research to find sources from scratch. Trigger this skill when the user wants to:
  turn documents/URLs/YouTube videos into podcast-style audio or video overviews; generate
  quizzes, flashcards, or study materials from their sources; create slide decks, infographics,
  or reports from research; use NotebookLM's built-in research to explore a topic without
  existing sources; share notebooks or export to Google Docs/Sheets; or any mention of
  "NotebookLM". Common phrasings: "make a podcast from this", "create study materials",
  "summarize these docs as audio", "generate a quiz from my PDF", "research this topic and
  make a presentation". Do NOT trigger for: building apps/UIs (React flashcard apps, d3.js
  visualizations), writing code (python-pptx scripts, podcast RSS parsers), audio file
  conversion (ffmpeg), Jupyter notebooks, or simple document reading/summarization that
  doesn't need NotebookLM's content generation.
---

# NotebookLM Skill

Generate rich AI content from documents using Google's NotebookLM via the `notebooklm-py` CLI.
This library provides programmatic access to NotebookLM features, including capabilities
unavailable in the web interface (batch downloads, editable PPTX, structured quiz export, etc.).

> **Important:** This uses undocumented Google APIs. Best for personal projects and prototyping.

## Prerequisites

The `notebooklm` CLI must be installed and authenticated. If not already set up:

```bash
pip install "notebooklm-py[browser]"
playwright install chromium
notebooklm login
```

Check auth status with `notebooklm status`. If expired, re-run `notebooklm login`.

## Core Workflow

Every task follows this pattern:

1. **Create or select a notebook** — each notebook is a container for sources and generated content
2. **Add sources** — URLs, files (PDF, DOCX, MD, images, audio/video), YouTube links, or pasted text
3. **Generate an artifact** — audio, video, quiz, slides, report, etc.
4. **Download the result** — in the user's preferred format

### Step 1: Notebook

```bash
# Create a new notebook
notebooklm create "Project Research"

# Or list existing ones and select
notebooklm list
notebooklm use <notebook_id>
```

Partial IDs work — you only need enough characters to be unique.

### Step 2: Sources

```bash
# Add various source types
notebooklm source add "https://example.com/article"
notebooklm source add "/path/to/document.pdf"
notebooklm source add "https://youtube.com/watch?v=..."
notebooklm source add --type text "Pasted content here"

# Add from Google Drive
notebooklm source add-drive "<drive_file_id>"

# AI-powered research (finds and adds sources automatically)
notebooklm source add-research "topic to research" --mode deep --wait --import-all

# List sources in current notebook
notebooklm source list
```

Multiple sources can be added — NotebookLM synthesizes across all of them.

### Step 3: Generate

All generate commands are async by default. Use `--wait` to block until complete.

```bash
# Audio podcast (the signature NotebookLM feature)
notebooklm generate audio --wait
notebooklm generate audio --format deep-dive --length long --wait
notebooklm generate audio --format brief --wait
notebooklm generate audio --format debate --wait
notebooklm generate audio --format critique --wait
notebooklm generate audio --language nl --wait    # 50+ languages supported

# Video overview
notebooklm generate video --wait
notebooklm generate video --format explainer --style whiteboard --wait

# Slide deck
notebooklm generate slide-deck --wait
notebooklm generate slide-deck --format presenter-slides --length short --wait

# Quiz
notebooklm generate quiz --wait
notebooklm generate quiz --difficulty hard --quantity more --wait

# Flashcards
notebooklm generate flashcards --wait
notebooklm generate flashcards --difficulty easy --wait

# Report
notebooklm generate report --wait
notebooklm generate report --format study-guide --wait
notebooklm generate report --format blog-post --wait
notebooklm generate report --format custom --instructions "Write a executive summary focusing on key metrics" --wait

# Infographic
notebooklm generate infographic --wait
notebooklm generate infographic --orientation landscape --detail detailed --wait

# Mind map (synchronous, no --wait needed)
notebooklm generate mind-map

# Data table
notebooklm generate data-table --instructions "Extract all dates and events mentioned" --wait
```

### Step 4: Download

```bash
# Audio/Video
notebooklm download audio output.mp3
notebooklm download video output.mp4

# Slide deck (PDF or editable PPTX)
notebooklm download slide-deck output.pdf
notebooklm download slide-deck output.pptx --format pptx

# Quiz & Flashcards (JSON, Markdown, or HTML)
notebooklm download quiz quiz.json --format json
notebooklm download quiz quiz.md --format markdown
notebooklm download flashcards cards.json --format json

# Other formats
notebooklm download mind-map mindmap.json
notebooklm download data-table data.csv
notebooklm download infographic infographic.png
```

## Chat & Questions

Ask questions about the sources in a notebook:

```bash
notebooklm ask "What are the main arguments in this paper?"
notebooklm ask "Compare the methodologies across all sources"

# View conversation history
notebooklm chat history
```

## Useful Patterns

### Full pipeline in one go

```bash
notebooklm create "Quarterly Review" && \
notebooklm source add report-q4.pdf && \
notebooklm source add report-q3.pdf && \
notebooklm generate audio --format deep-dive --wait && \
notebooklm download audio quarterly-podcast.mp3
```

### Generate multiple artifact types

```bash
notebooklm generate audio --wait &
notebooklm generate quiz --wait &
notebooklm generate report --format study-guide --wait &
wait
notebooklm download audio overview.mp3
notebooklm download quiz quiz.json --format json
```

### Research a topic from scratch

```bash
notebooklm create "AI Safety Research"
notebooklm source add-research "recent developments in AI alignment" --mode deep --wait --import-all
notebooklm generate audio --format deep-dive --wait
notebooklm download audio ai-safety-podcast.mp3
```

### Revise individual slides

```bash
notebooklm artifact list
notebooklm slide revise <artifact_id> <slide_number> "Make the title more concise and add a bullet about ROI"
```

## Audio Formats Explained

| Format | Description |
|--------|-------------|
| `deep-dive` | Two hosts explore the topic in depth (default, classic NotebookLM style) |
| `brief` | Shorter, single-host summary |
| `critique` | Critical analysis with counterpoints |
| `debate` | Two hosts argue different perspectives |

Lengths: `short`, `default`, `long`

## Video Styles

`classic`, `whiteboard`, `kawaii`, `anime`, `watercolor`, `retro-print`, `heritage`, `paper-craft`, `auto-select`

## Troubleshooting

- **"Session expired"**: Run `notebooklm login` again
- **Generation stuck**: Check `notebooklm artifact list` for status. Artifacts show status 1 (processing), 2 (pending), 3 (completed)
- **Rate limiting**: Add short delays between operations if doing batch work

## Reference

For the complete CLI reference and Python API, see:
- `references/cli-reference.md` — all CLI commands, flags, and options
- `references/python-api.md` — async Python API for complex/programmatic workflows

Use the Python API (via a script) when you need conditional logic, error handling,
or complex multi-step workflows that go beyond what chained CLI commands can do.
