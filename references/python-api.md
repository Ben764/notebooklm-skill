# NotebookLM Python API Reference

Use the Python API when you need conditional logic, error handling, or complex multi-step
workflows that go beyond chained CLI commands. Write async Python scripts.

## Table of Contents

1. [Client Setup](#client-setup)
2. [Notebooks](#notebooks)
3. [Sources](#sources)
4. [Artifacts](#artifacts)
5. [Chat](#chat)
6. [Research](#research)
7. [Notes](#notes)
8. [Settings](#settings)
9. [Sharing](#sharing)
10. [Data Types](#data-types)
11. [Enums](#enums)

## Client Setup

```python
import asyncio
from notebooklm import NotebookLMClient

async def main():
    async with await NotebookLMClient.from_storage() as client:
        # All API calls go here
        notebooks = await client.notebooks.list()
        print(notebooks)

asyncio.run(main())
```

Authentication sources (in order of precedence):
1. Explicit path passed to `from_storage(path=...)`
2. `NOTEBOOKLM_AUTH_JSON` environment variable
3. `NOTEBOOKLM_HOME/storage_state.json`
4. `~/.notebooklm/storage_state.json`

## Notebooks

```python
# List all notebooks
notebooks = await client.notebooks.list()
# Returns: list[Notebook] with id, title, created_at, sources_count, is_owner

# Create
notebook = await client.notebooks.create("My Notebook")

# Rename
await client.notebooks.rename(notebook_id, "New Name")

# Delete
await client.notebooks.delete(notebook_id)

# Get description (parsed summary + suggested topics)
desc = await client.notebooks.get_description(notebook_id)

# Get summary (raw text)
summary = await client.notebooks.get_summary(notebook_id)
```

## Sources

```python
# Add sources
source = await client.sources.add_url(notebook_id, "https://example.com", wait=True)
source = await client.sources.add_file(notebook_id, "/path/to/file.pdf")
source = await client.sources.add_youtube(notebook_id, "https://youtube.com/watch?v=...")
source = await client.sources.add_text(notebook_id, "Pasted content here")
source = await client.sources.add_drive(notebook_id, "drive_file_id")

# List sources
sources = await client.sources.list(notebook_id)
# Returns: list[Source] with id, title, url, created_at, .kind property

# Get content
fulltext = await client.sources.get_fulltext(notebook_id, source_id)
# Returns: SourceFulltext with source_id, title, content, url, char_count

guide = await client.sources.get_guide(notebook_id, source_id)

# Refresh
await client.sources.refresh(notebook_id, source_id)
```

## Artifacts

### Generation

All generation methods return a status object with a `task_id` for async tracking.

```python
from notebooklm import (
    AudioFormat, AudioLength, VideoFormat, VideoStyle,
    QuizDifficulty, QuizQuantity, ReportFormat,
    InfographicOrientation, InfographicDetail,
    SlideDeckFormat, SlideDeckLength
)

# Audio
status = await client.artifacts.generate_audio(
    notebook_id,
    instructions="Focus on the key findings",  # optional
    format=AudioFormat.DEEP_DIVE,               # DEEP_DIVE, BRIEF, CRITIQUE, DEBATE
    length=AudioLength.DEFAULT,                  # SHORT, DEFAULT, LONG
    language="en"                                # ISO code, 50+ languages
)

# Video
status = await client.artifacts.generate_video(
    notebook_id,
    format=VideoFormat.EXPLAINER,    # EXPLAINER, BRIEF
    style=VideoStyle.WHITEBOARD      # AUTO_SELECT, CLASSIC, WHITEBOARD, KAWAII, etc.
)

# Quiz
status = await client.artifacts.generate_quiz(
    notebook_id,
    difficulty=QuizDifficulty.HARD,   # EASY, MEDIUM, HARD
    quantity=QuizQuantity.STANDARD     # FEWER, STANDARD
)

# Flashcards
status = await client.artifacts.generate_flashcards(
    notebook_id,
    difficulty=QuizDifficulty.MEDIUM,
    quantity=QuizQuantity.STANDARD
)

# Slide deck
status = await client.artifacts.generate_slide_deck(
    notebook_id,
    format=SlideDeckFormat.DETAILED_DECK,  # DETAILED_DECK, PRESENTER_SLIDES
    length=SlideDeckLength.DEFAULT          # DEFAULT, SHORT
)

# Report
status = await client.artifacts.generate_report(
    notebook_id,
    template=ReportFormat.STUDY_GUIDE,  # BRIEFING_DOC, STUDY_GUIDE, BLOG_POST, CUSTOM
    instructions="Custom prompt here"    # required for CUSTOM
)

# Infographic
status = await client.artifacts.generate_infographic(
    notebook_id,
    orientation=InfographicOrientation.LANDSCAPE,  # LANDSCAPE, PORTRAIT, SQUARE
    detail_level=InfographicDetail.DETAILED         # CONCISE, STANDARD, DETAILED
)

# Mind map (synchronous)
await client.artifacts.generate_mind_map(notebook_id)

# Data table
status = await client.artifacts.generate_data_table(
    notebook_id,
    instructions="Extract all dates and events"
)
```

### Waiting for Completion

```python
result = await client.artifacts.wait_for_completion(
    notebook_id,
    task_id,
    timeout=300,      # seconds, default 300
    poll_interval=5   # seconds between checks
)
# Returns: GenerationStatus with is_completed, url
```

### Downloads

```python
path = await client.artifacts.download_audio(notebook_id, "output.mp3")
path = await client.artifacts.download_video(notebook_id, "output.mp4")
path = await client.artifacts.download_slide_deck(notebook_id, "output.pptx", format="pptx")
path = await client.artifacts.download_quiz(notebook_id, "quiz.json", output_format="json")
path = await client.artifacts.download_flashcards(notebook_id, "cards.md", output_format="markdown")
path = await client.artifacts.download_mind_map(notebook_id, "mindmap.json")
path = await client.artifacts.download_data_table(notebook_id, "data.csv")
```

### Export to Google Docs/Sheets

```python
from notebooklm import ExportType

await client.artifacts.export(notebook_id, artifact_id, ExportType.DOCS)
await client.artifacts.export(notebook_id, artifact_id, ExportType.SHEETS)
```

## Chat

```python
# Ask a question
result = await client.chat.ask(notebook_id, "What are the key findings?")
# Returns: AskResult with:
#   .answer — text with inline citations [1], [2]
#   .conversation_id — for follow-up questions
#   .turn_number
#   .references — list[ChatReference] with source_id, citation_number, cited_text

# Follow-up question (same conversation)
result2 = await client.chat.ask(
    notebook_id,
    "Can you elaborate on point 2?",
    conversation_id=result.conversation_id
)

# Get history
history = await client.chat.get_history(notebook_id)

# Configure persona
from notebooklm import ChatGoal, ChatResponseLength
await client.chat.configure(
    notebook_id,
    goal=ChatGoal.CUSTOM,
    custom_prompt="You are a critical reviewer",
    response_length=ChatResponseLength.LONGER
)
```

### Citation Resolution

```python
# Get fulltext and find citation context
fulltext = await client.sources.get_fulltext(notebook_id, source_id)
context = fulltext.find_citation_context(cited_text)
```

## Research

```python
# Start research
task_id = await client.research.start(notebook_id, "AI alignment", mode="deep")

# Poll status
status = await client.research.poll(notebook_id, task_id)

# Import discovered sources
await client.research.import_sources(notebook_id, task_id)
```

## Notes

```python
await client.notes.create(notebook_id, "Note content here")
notes = await client.notes.list(notebook_id)
await client.notes.update(notebook_id, note_id, "Updated content")
await client.notes.delete(notebook_id, note_id)

# Mind maps stored as notes
mind_maps = await client.notes.list_mind_maps(notebook_id)
```

## Settings

```python
lang = await client.settings.get_language()
await client.settings.set_language("nl")  # Dutch
```

## Sharing

```python
from notebooklm import ShareAccess, ShareViewLevel, SharePermission

# Get status
status = await client.sharing.get_status(notebook_id)
# Returns: ShareStatus with is_public, access, view_level, shared_users, share_url

# Enable public sharing
await client.sharing.enable(notebook_id, access=ShareAccess.ANYONE_WITH_LINK)
await client.sharing.set_view_level(notebook_id, ShareViewLevel.CHAT_ONLY)

# Add users
await client.sharing.add_user(notebook_id, "user@example.com", SharePermission.EDITOR)
await client.sharing.remove_user(notebook_id, "user@example.com")

# Disable
await client.sharing.disable(notebook_id)
```

## Data Types

| Type | Fields |
|------|--------|
| `Notebook` | id, title, created_at, sources_count, is_owner |
| `Source` | id, title, url, created_at, `.kind` → `SourceType` |
| `Artifact` | id, title, status (1=processing, 2=pending, 3=completed), created_at, url, `.kind` → `ArtifactType`, `.is_completed`, `.is_quiz`, `.is_flashcards` |
| `AskResult` | answer, conversation_id, turn_number, is_follow_up, references, raw_response |
| `ChatReference` | source_id, citation_number, cited_text, start_char, end_char, chunk_id |
| `SourceFulltext` | source_id, title, content, url, char_count, `.find_citation_context()` |
| `ShareStatus` | notebook_id, is_public, access, view_level, shared_users, share_url |
| `SharedUser` | email, permission, display_name, avatar_url |

## Enums

| Enum | Values |
|------|--------|
| `AudioFormat` | DEEP_DIVE, BRIEF, CRITIQUE, DEBATE |
| `AudioLength` | SHORT, DEFAULT, LONG |
| `VideoFormat` | EXPLAINER, BRIEF |
| `VideoStyle` | AUTO_SELECT, CUSTOM, CLASSIC, WHITEBOARD, KAWAII, ANIME, WATERCOLOR, RETRO_PRINT, HERITAGE, PAPER_CRAFT |
| `QuizDifficulty` | EASY, MEDIUM, HARD |
| `QuizQuantity` | FEWER, STANDARD |
| `ReportFormat` | BRIEFING_DOC, STUDY_GUIDE, BLOG_POST, CUSTOM |
| `InfographicOrientation` | LANDSCAPE, PORTRAIT, SQUARE |
| `InfographicDetail` | CONCISE, STANDARD, DETAILED |
| `SlideDeckFormat` | DETAILED_DECK, PRESENTER_SLIDES |
| `SlideDeckLength` | DEFAULT, SHORT |
| `ExportType` | DOCS (1), SHEETS (2) |
| `ShareAccess` | RESTRICTED, ANYONE_WITH_LINK |
| `ShareViewLevel` | FULL_NOTEBOOK, CHAT_ONLY |
| `SharePermission` | OWNER, EDITOR, VIEWER |
| `SourceType` | GOOGLE_DOCS, GOOGLE_SLIDES, GOOGLE_SPREADSHEET, PDF, PASTED_TEXT, WEB_PAGE, YOUTUBE, MARKDOWN, DOCX, CSV, IMAGE, MEDIA, UNKNOWN |
| `ArtifactType` | AUDIO, VIDEO, REPORT, QUIZ, FLASHCARDS, MIND_MAP, INFOGRAPHIC, SLIDE_DECK, DATA_TABLE, UNKNOWN |
| `SourceStatus` | PROCESSING, READY, ERROR, PREPARING |
| `ChatGoal` | DEFAULT, CUSTOM, LEARNING_GUIDE |
| `ChatResponseLength` | DEFAULT, LONGER, SHORTER |

## Error Handling

The API raises `RPCError` for failures. Auth errors (401/403) trigger automatic token refresh.
For rate limiting, add ~2 second delays between operations and implement exponential backoff.

```python
from notebooklm.exceptions import RPCError

try:
    await client.artifacts.generate_audio(notebook_id)
except RPCError as e:
    print(f"API error: {e}")
```
