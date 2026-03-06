# NotebookLM CLI Reference

Complete reference for the `notebooklm` command-line tool.

## Table of Contents

1. [Global Options](#global-options)
2. [Authentication](#authentication)
3. [Notebooks](#notebooks)
4. [Sources](#sources)
5. [Generation](#generation)
6. [Downloads](#downloads)
7. [Artifacts](#artifacts)
8. [Chat](#chat)
9. [Research](#research)
10. [Notes](#notes)
11. [Sharing](#sharing)
12. [Settings](#settings)

## Global Options

```
notebooklm [--storage PATH] [--version] <command> [OPTIONS] [ARGS]
```

| Option | Description |
|--------|-------------|
| `--storage PATH` | Override default storage path (`~/.notebooklm/storage_state.json`) |
| `--version` | Show version |

Environment variables:
- `NOTEBOOKLM_HOME` — override config directory
- `NOTEBOOKLM_AUTH_JSON` — provide auth as JSON string (CI/CD)
- `NOTEBOOKLM_DEBUG_RPC` — enable RPC debug logging

## Authentication

```bash
notebooklm login              # Browser-based login
notebooklm status             # Check auth status and current notebook
notebooklm auth-diagnostics   # Detailed auth debugging
```

## Notebooks

```bash
notebooklm create "<name>"           # Create new notebook (auto-selects it)
notebooklm list                      # List all notebooks
notebooklm use <notebook_id>         # Switch active notebook
notebooklm rename <id> "<new_name>"  # Rename notebook
notebooklm delete <id>               # Delete notebook
```

Partial IDs work for all commands — provide enough characters to be unique.

## Sources

```bash
# Add sources
notebooklm source add "<url_or_filepath>"       # Auto-detects type
notebooklm source add --type text "<content>"    # Pasted text
notebooklm source add-drive "<drive_file_id>"    # Google Drive file
notebooklm source add-youtube "<youtube_url>"    # YouTube video

# Manage sources
notebooklm source list                           # List all sources
notebooklm source rename <id> "<new_title>"      # Rename source
notebooklm source refresh <id>                   # Re-index source
notebooklm source guide <id>                     # AI-generated summary
notebooklm source fulltext <id>                  # Full indexed content

# AI Research
notebooklm source add-research "<query>" [OPTIONS]
  --mode fast|deep          # Research depth (default: fast)
  --wait                    # Block until research completes
  --no-wait                 # Return immediately with task ID
  --import-all              # Auto-import discovered sources
```

Supported file types: PDF, DOCX, MD, TXT, CSV, images (OCR), audio, video

## Generation

All generation commands accept `--wait` to block until completion.
Without `--wait`, they return a task ID for polling.

### Audio

```bash
notebooklm generate audio [OPTIONS] [--wait]
  --format deep-dive|brief|critique|debate   # Default: deep-dive
  --length short|default|long                # Default: default
  --language <code>                          # ISO language code (e.g., nl, de, ja)
  --instructions "<custom_instructions>"     # Guide the content
```

### Video

```bash
notebooklm generate video [OPTIONS] [--wait]
  --format explainer|brief
  --style auto-select|classic|whiteboard|kawaii|anime|watercolor|retro-print|heritage|paper-craft|custom
```

### Slide Deck

```bash
notebooklm generate slide-deck [OPTIONS] [--wait]
  --format detailed-deck|presenter-slides
  --length default|short
```

### Quiz

```bash
notebooklm generate quiz [OPTIONS] [--wait]
  --difficulty easy|medium|hard    # Default: medium
  --quantity fewer|standard        # Default: standard
```

### Flashcards

```bash
notebooklm generate flashcards [OPTIONS] [--wait]
  --difficulty easy|medium|hard
  --quantity fewer|standard
```

### Report

```bash
notebooklm generate report [OPTIONS] [--wait]
  --format briefing-doc|study-guide|blog-post|custom
  --instructions "<prompt>"        # Required for custom format
```

### Infographic

```bash
notebooklm generate infographic [OPTIONS] [--wait]
  --orientation landscape|portrait|square
  --detail concise|standard|detailed
```

### Mind Map

```bash
notebooklm generate mind-map      # Synchronous, no --wait needed
```

### Data Table

```bash
notebooklm generate data-table [OPTIONS] [--wait]
  --instructions "<what to extract>"
```

## Downloads

```bash
notebooklm download audio <filepath>                    # .mp3
notebooklm download video <filepath>                    # .mp4
notebooklm download slide-deck <filepath> [--format pdf|pptx]
notebooklm download quiz <filepath> [--format json|markdown|html]
notebooklm download flashcards <filepath> [--format json|markdown|html]
notebooklm download mind-map <filepath>                 # .json
notebooklm download data-table <filepath>               # .csv
notebooklm download infographic <filepath>              # .png
```

## Artifacts

```bash
notebooklm artifact list                  # List all artifacts in notebook
notebooklm artifact wait <task_id>        # Wait for generation to complete
notebooklm artifact rename <id> "<name>"  # Rename artifact
notebooklm artifact delete <id>           # Delete artifact

# Slide-specific
notebooklm slide revise <artifact_id> <slide_number> "<instructions>"
```

Artifact status codes: 1 = processing, 2 = pending, 3 = completed

## Chat

```bash
notebooklm ask "<question>"                         # Ask about sources
notebooklm chat history                             # View conversation
notebooklm chat configure --goal learning-guide     # Set persona
notebooklm chat configure --goal custom --prompt "<instructions>"
notebooklm chat configure --response-length shorter|default|longer
```

## Research

```bash
notebooklm research status <task_id>           # Check research progress
notebooklm research wait <task_id>             # Wait for completion
notebooklm research wait <task_id> --import-all  # Wait and import sources
```

## Notes

```bash
notebooklm note add "<content>"        # Create a note
notebooklm note list                   # List all notes
notebooklm note update <id> "<content>"
notebooklm note delete <id>
notebooklm chat save-as-note           # Save last chat as note
```

## Sharing

```bash
notebooklm share status                              # View sharing settings
notebooklm share enable --access anyone               # Enable public link
notebooklm share enable --view-level chat-only        # Chat-only access
notebooklm share disable                              # Make private
notebooklm share add-user "<email>" --permission viewer|editor
notebooklm share remove-user "<email>"
```

## Settings

```bash
notebooklm settings get-language         # Current output language
notebooklm settings set-language <code>  # Set output language (e.g., nl, en, ja)
```
