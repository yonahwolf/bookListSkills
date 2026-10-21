# First-time setup

Run this when `config.json` does not exist, or when the user asks to change podcasts or the library. Keep it conversational and short: one question at a time, with sensible defaults offered. Nothing is preloaded, so every podcast and library comes from the user.

If a config already exists, show a one-paragraph summary of it and ask what to change rather than starting over.

## 1. Podcasts (required)

Ask: "Which podcasts do you want me to watch for author-guests?" Accept any number, in any form (names, Apple/Spotify links, RSS feeds).

For each podcast:

1. Search for the show and confirm the exact one with the user if the name is ambiguous (several shows share common names).
2. Find the most reliable episode list. Prefer the **RSS feed**: it has exact dates, is rarely blocked, and is often more complete than the show's website. Apple's public lookup returns it: `https://itunes.apple.com/search?media=podcast&term=<show name>` (use the `feedUrl` of the result whose `artistName` matches the publisher). Fall back to the official site's archive, then Apple Podcasts or Spotify pages, then an aggregator.
3. Fetch it once to check that it lists episode dates and guest names. If it is blocked or doesn't, try the next source.
4. Note anything a later run should know, such as a feed that also carries audiobook excerpts or a spin-off show, or a feed published under a different name than the show.

Save each as `{ "name", "archive_url", "archive_type": "rss|web", "notes", "added" }`. The user can add or remove podcasts later just by asking.

## 2. Preferences (optional, offer defaults)

Ask these together in one message, with defaults the user can accept by saying "defaults are fine":

- **Topics to prioritise** (default: none, all topics).
- **Default timeframe** for manual runs (default: last 3 months).
- **Amazon purchase links** (default: yes).

## 3. Libraries (optional)

Ask: "Do you want me to check a library for these books? If so, which library system?" The user may name more than one; set each up separately and save them in the `libraries` list. If the user declines, set `libraries` to `[]` and skip to step 4.

A town library is often part of a county or regional system that runs the shared catalog and Libby collection (for example, a city library inside a county library system). A city name can also map to several separate systems; if so, ask which one.

For each library, work out the details yourself rather than asking:

1. **Catalog.** Search for the library's online catalog and identify the vendor (BiblioCommons, Polaris, Sierra/Vega, Aspen Discovery, Evergreen, CARL•Connect, Koha, Apollo, or another). Save `catalog_url`, `catalog_system`, and a `catalog_search_url` template with `{query}` in it. Test one search for a recent book and record in `catalog_automated` whether the catalog can be read automatically (true) or blocks automated access (false).
2. **Digital lending.** Find which platforms the library offers (Libby/OverDrive, Hoopla, cloudLibrary, Palace, Boundless) and save each with its URL.
3. **Formats.** Ask which formats the user wants (print, ebook, audiobook; default all). Ask once for all libraries.
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
