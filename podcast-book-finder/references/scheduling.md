# Running on a schedule

A scheduled run checks the configured podcasts for episodes since the last run, reports only new books, and handles holds according to `hold_mode`. Setup must be done first, interactively, because a scheduled run never asks questions.

## Setting it up for the user

1. Make sure `config.json` exists. If not, run setup first.
2. Ask how often (default: weekly, Monday morning). Weekly or every two weeks suits most podcasts; daily rarely finds anything new.
3. If `hold_mode` is `"auto"`, confirm it once more: holds will be placed without asking, up to `max_auto_holds_per_run` per run.
4. Create the schedule:
   - **If you have a scheduling tool** in this environment (for example a scheduled-tasks tool in the Claude desktop app, or automations in the Codex app), create the task yourself with the prompt below, after showing the user the schedule and prompt.
   - **Otherwise** give the user the matching steps from the platform sections below.
5. Save `schedule` in the config (cadence, platform, created date).

Use this prompt for every platform, filling in nothing else; everything it needs is in the config:

```text
Scheduled run of the podcast-book-finder skill. Use the saved config and history,
report only books that are new since the last run, follow the configured hold_mode,
and do not ask any questions. Save the list and give me a short summary.
```

## Claude

### Claude desktop app (recommended)

Scheduled tasks run on the user's computer, so they can read `~/.podcast-book-finder/` and use any MCP servers connected there, including a hold server.

1. Install the skill: copy the `podcast-book-finder` folder to `~/.claude/skills/` (or upload it as a skill under Settings → Capabilities in the app).
2. Open the app and ask: *"Run the podcast book finder every Monday at 8am as a scheduled task."* Claude creates the task with the prompt above. Alternatively, create it from the scheduled tasks list in the sidebar and paste the prompt.
3. The computer must be awake and the app running at the scheduled time; missed runs catch up on the next one because the window is "since last run".

### Claude Code CLI with cron or launchd

For a headless machine. Install the skill to `~/.claude/skills/podcast-book-finder/`, then schedule:

```bash
claude -p "Scheduled run of the podcast-book-finder skill. Use the saved config and history, report only books that are new since the last run, follow the configured hold_mode, and do not ask any questions." --allowedTools "WebSearch WebFetch Read Write Edit"
```

Add the hold server's tools to `--allowedTools` (for example `mcp__<server-name>__*`) if `hold_mode` is `"auto"`. Example crontab line for Mondays at 8:00:

```text
0 8 * * 1 cd ~ && /path/to/claude -p "..." --allowedTools "..." >> ~/.podcast-book-finder/cron.log 2>&1
```

### Claude Code cloud routines (`/schedule`)

Cloud routines run in a fresh cloud environment, not on the user's machine, so they cannot see `~/.podcast-book-finder/` or locally installed MCP servers. To use them, keep the config, history and lists in a private Git repository, set `PODCAST_BOOK_FINDER_HOME` to that checkout in the routine's environment, and have the routine commit the updated `history.json` and list. Holds work only with a remote MCP connector. Prefer the desktop app unless the user specifically wants this.

## OpenAI

### Codex app automations (recommended)

1. Install the skill: copy the `podcast-book-finder` folder to `~/.codex/skills/` (or `$CODEX_HOME/skills/`).
2. Run setup once in a Codex thread: *"$podcast-book-finder set up"*.
3. Create an automation in the Codex app with the prompt above (prefixed with `$podcast-book-finder`) and a weekly schedule. Results land in the automations inbox.
4. Make sure the automation's permissions allow web search, network access, and writing to `~/.podcast-book-finder/`.

### Codex CLI with cron

```bash
codex exec "\$podcast-book-finder Scheduled run. Use the saved config and history, report only books that are new since the last run, follow the configured hold_mode, and do not ask any questions."
```

Schedule it with cron or launchd as in the Claude CLI example. Web search and write access to `~/.podcast-book-finder/` must be enabled in the Codex configuration or flags for non-interactive runs; check `codex exec --help` for the current options.

### ChatGPT scheduled tasks

ChatGPT has no file storage for this skill, so use the portable prompt in [book_list_prompt.md](../../book_list_prompt.md):

1. Run setup in a chat with the portable prompt. At the end, ChatGPT prints a config block.
2. Ask ChatGPT to create a task, for example: *"Every Monday at 8am, run the Podcast Book Finder with this config and the instructions above"*, and include the prompt and config in the task. Manage it from the Scheduled page in ChatGPT.
3. Because tasks don't keep a reliable history, the task uses "since the last 7 days" (or the cadence) as its window instead of `last_run`.

Scheduled ChatGPT tasks never place holds; they list hold candidates and links, and the user can place holds in a regular chat (through a library connector, if one is added) or on the library's site.
