# 📚 Podcast Book Finder

**Turn the guest lists of the podcasts you follow into a reading list, with library availability, optional holds, and Amazon links. Run it on demand or on a schedule.**

Created by [Yonah Wolf](https://yonah.name)

---

## What it does

- **First-time setup** asks which podcasts you follow and, optionally, which library you use. Nothing is preloaded; it finds each show's episode archive and your library's catalog and digital lending platforms (Libby/OverDrive, Hoopla, and others) itself.
- **Finds author-guests** in recent episodes and the books they wrote.
- **Checks your library** for print, ebook and audiobook availability.
- **Places holds** when your library can be reached through an MCP server. During setup it looks for one (already-connected tools, your AI app's connector directory, the MCP registry and the web) and tells you what it found. It never installs anything or handles your library card itself, and by default it asks before every hold.
- **Remembers what it has shown you**, so each run reports only new books.
- **Runs on a schedule** (weekly by default) in Claude, Codex or ChatGPT.
- **Optional Amazon links** for each book.

## Contents

| Path | For | What it is |
|------|-----|------------|
| [`podcast-book-finder/`](podcast-book-finder/) | **Claude** and **Codex** | An Agent Skill: `SKILL.md` plus reference files for setup, library holds and scheduling, and an example config. |
| [`book_list_prompt.md`](book_list_prompt.md) | **ChatGPT, Gemini, or any assistant** | A single portable prompt with the same setup and run flow. Paste it into a chat, project or custom GPT. |
| [`airline/`](airline/) | Claude | A separate flight-search skill. |

## Install and run

### Claude (desktop app or Claude Code)

```bash
cp -R podcast-book-finder ~/.claude/skills/
```

Then ask: *"Set up the podcast book finder."* After setup: *"Run my podcast book list"*, or *"Run the podcast book finder every Monday at 8am."*

In the Claude web or desktop app you can also upload the `podcast-book-finder` folder (zipped) as a skill under **Settings → Capabilities**.

### Codex (app or CLI)

```bash
cp -R podcast-book-finder ~/.codex/skills/
```

Then: *"$podcast-book-finder set up"*, and later *"$podcast-book-finder run"*.

### ChatGPT and other assistants

Paste [`book_list_prompt.md`](book_list_prompt.md) into a new chat (or a project's instructions) and follow the setup questions. Save the config block it prints at the end and paste it in next time.

## Scheduling

Full instructions are in [`podcast-book-finder/references/scheduling.md`](podcast-book-finder/references/scheduling.md). In short:

| Platform | How |
|----------|-----|
| Claude desktop app | Ask Claude to create a scheduled task, or add one from the scheduled tasks list. Runs on your computer, with access to your config and a local hold server. |
| Claude Code CLI | `claude -p "<scheduled-run prompt>"` from cron or launchd. |
| Codex app | Create an automation whose prompt starts with `$podcast-book-finder`. |
| Codex CLI | `codex exec "<scheduled-run prompt>"` from cron or launchd. |
| ChatGPT | Ask ChatGPT to create a scheduled task with the portable prompt and your config. |

Scheduled runs never ask questions: they report books new since the last run, and place holds only if you chose automatic holds during setup (capped per run).

## Your data

On Claude and Codex, everything lives in `~/.podcast-book-finder/` (override with `PODCAST_BOOK_FINDER_HOME`):

- `config.json`: your podcasts, library and preferences ([example](podcast-book-finder/config.example.json))
- `history.json`: books already reported, and any holds placed
- `lists/`: generated reading lists

Library card numbers and PINs are never stored here; they stay in the hold server's own configuration.

## Disclosure

Book links to Amazon are affiliate links. As an Amazon Associate, I earn from qualifying purchases. This does not affect the price you pay.

## License

This project is licensed under the [GNU General Public License v3.0](LICENSE). You're free to use, modify, and distribute it, but any derivative work must also be released under GPLv3 and include the same freedoms. See the LICENSE file for full details.
