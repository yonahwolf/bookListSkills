# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Prompt-only repo (no code) with shareable AI skills, licensed GPLv3:

- `SKILL.md` — **Podcast Book Finder** Claude skill (frontmatter `name: podcast-book-finder`). Given podcast(s) and a timeframe, it finds guests who are authors, lists their books, and can check library availability (BiblioCommons, CARL•Connect, Vega, Evergreen, OverDrive/Libby, Hoopla) and add Amazon links.
- `book_list_prompt.md` — the same workflow as a portable copy-paste prompt for ChatGPT, Gemini, etc. The README calls this file `PROMPT_PORTABLE.md`; the actual filename is `book_list_prompt.md`.
- `airline/SKILL_airline.md` — separate **flight-search** skill with a user-editable YAML config block of cards, transfer partners, and loyalty programs.

## Conventions

- `SKILL.md` and `book_list_prompt.md` are two versions of one workflow. When you change one (phases, supported podcasts, library systems, output format), make the matching change in the other and in the README lists.
- The Amazon affiliate tag is `theroadtoshod-20` and must appear in every Amazon link format in both files.
- Keep the skill frontmatter `description` trigger-rich; it decides when Claude loads the skill.
- The installed copy of the podcast skill in this Claude environment is `anthropic-skills:podcast-book-finder`. Edits here don't update it automatically.
