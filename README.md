# NotebookLM Skill

A Claude Code skill for generating content with Google NotebookLM — podcasts, videos, quizzes, flashcards, slide decks, and more from your documents and URLs.

## About

This skill was built as a learning exercise, following the [Claude Code skill creation tutorial](https://docs.anthropic.com/en/docs/claude-code/skills). It serves as a practical example of how to create and structure a Claude Code skill.

## What it does

When invoked, the skill instructs Claude to use the `notebooklm-py` CLI to:

- Turn documents, URLs, and YouTube videos into podcast-style audio or video overviews
- Generate quizzes, flashcards, and study materials from sources
- Create slide decks, infographics, reports, and more
- Use NotebookLM's built-in AI research to explore topics from scratch

## Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI
- [notebooklm-py](https://github.com/nichochar/notebooklm-py) installed and authenticated

## Installation

```bash
mkdir -p ~/.claude/skills/notebooklm
cp SKILL.md ~/.claude/skills/notebooklm/SKILL.md
```

Then use `/notebooklm` in Claude Code to invoke the skill.
