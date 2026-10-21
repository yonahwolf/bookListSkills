# First-time setup

Run this when `config.json` does not exist, or when the user asks to change podcasts or the library. Keep it conversational and short: one question at a time, with sensible defaults offered. Nothing is preloaded, so every podcast and library comes from the user.

If a config already exists, show a one-paragraph summary of it and ask what to change rather than starting over.

## 1. Podcasts (required)

Ask: "Which podcasts do you want me to watch for author-guests?" Accept any number, in any form (names, Apple/Spotify links, RSS feeds).

For each podcast:

1. Search for the show and confirm the exact one with the user if the name is ambiguous (several shows share common names).
2. Find the most reliable episode list: the official site's archive, then the RSS feed, then Apple Podcasts or Spotify, then an aggregator. Save it as `archive_url`.
3. Fetch it once to check that it lists episode dates and guest names. If it doesn't, try the next source.

Save each as `{ "name", "archive_url", "added" }`. The user can add or remove podcasts later just by asking.

## 2. Preferences (optional, offer defaults)

Ask these together in one message, with defaults the user can accept by saying "defaults are fine":

- **Topics to prioritise** (default: none, all topics).
- **Default timeframe** for manual runs (default: last 3 months).
- **Amazon purchase links** (default: yes).

## 3. Library (optional)

Ask: "Do you want me to check a library for these books? If so, which library system?" If the user declines, set `library` to `null` and skip to step 4.

Otherwise, work out the details yourself rather than asking:

1. **Catalog.** Search for the library's online catalog and identify the vendor (BiblioCommons, Polaris, Sierra/Vega, Aspen Discovery, Evergreen, CARL•Connect, Koha, Apollo, or another). Save `catalog_url` and `catalog_system`. Test one search to see whether the catalog can be read automatically or blocks automated access.
2. **Digital lending.** Find which platforms the library offers (Libby/OverDrive, Hoopla, cloudLibrary, Palace, Boundless) and save each with its URL.
3. **Formats.** Ask which formats the user wants (print, ebook, audiobook; default all).
4. **Holds.** Follow "Discovering a hold MCP server" in [library-holds.md](library-holds.md). Report what you found in plain terms, for example: "There's a community MCP server for BiblioCommons libraries that can place holds. It's third-party, so review it before installing. Want the install steps?" or "I couldn't find an MCP server that can place holds at this library, so I'll include hold links you can click instead."
5. **Hold mode.** If a hold server is connected and verified, ask how holds should work:
   - `ask` (default): suggest holds and place only the ones the user approves.
   - `auto`: on scheduled runs, place holds automatically, up to `max_auto_holds_per_run` (default 3), only for books with no copy available.
   - `off`: never place holds.

Never ask for or store a library card number or PIN. Sign-in belongs to the MCP server's own configuration.

## 4. Save and offer next steps

1. Write `config.json` (create `~/.podcast-book-finder/` and `lists/` if missing) and an empty `history.json` (`[]`). On a platform without file access, give the user the JSON to paste into their project instructions or task prompt, and explain where.
2. Show a short summary: podcasts, library and hold status, preferences.
3. Offer two next steps: "Run it now?" and "Run it on a schedule?" (see [scheduling.md](scheduling.md)).
