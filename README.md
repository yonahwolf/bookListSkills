# 📚 Podcast Book Finder

**Turn any podcast's guest list into a curated reading list — with library availability and Amazon purchase links.**

Created by [Yonah Wolf](https://yonah.name)

---

## What It Does

You give it a podcast name and a timeframe. It searches the web, identifies which guests are authors, finds their books, and produces a clean reading list. Optionally, it can:

- **Check your local library** for availability (physical + digital via Libby/OverDrive)
- **Include Amazon purchase links** for each book

It works with any interview-format podcast — Joe Rogan, Huberman Lab, Diary of a CEO, Tim Ferriss, WHOOP, Danger Close, or anything else.

## Two Versions

| File | Best For | How to Use |
|------|----------|------------|
| `SKILL.md` | **Claude** (Anthropic) | Save to your Claude skills directory at `/mnt/skills/user/podcast-book-finder/SKILL.md`. Claude will auto-discover it whenever you ask for podcast book recommendations. |
| `PROMPT_PORTABLE.md` | **ChatGPT, Gemini, Claude, or any AI** | Copy-paste the entire file as your first message in a new conversation. Then follow up with your request. |

## Example Prompts

Once the skill/prompt is loaded, just ask naturally:

> *"Find me health and nutrition books from Huberman Lab guests in 2024-2025."*

> *"What books have Joe Rogan's guests written this year? Check the Chicago Public Library."*

> *"Give me a reading list from Diary of a CEO and Mel Robbins Podcast guests with Amazon links."*

> *"Find books from Danger Close podcast guests that are available at the LA Public Library. Include Amazon links."*

## Supported Podcasts

Works with any podcast, but these are pre-loaded with archive URLs for faster results:

- The Joe Rogan Experience
- The Diary of a CEO (Steven Bartlett)
- The Mel Robbins Podcast
- Call Her Daddy (Alex Cooper)
- This Past Weekend (Theo Von)
- Huberman Lab
- WHOOP Podcast
- The Drive (Peter Attia)
- Rich Roll Podcast
- Tim Ferriss Show
- The Shawn Ryan Show
- Danger Close (Jack Carr)
- Found My Fitness (Rhonda Patrick)
- On Purpose (Jay Shetty)

## Library Systems

The skill knows how to search these catalog systems (and will auto-detect others):

- **BiblioCommons** (Chicago PL, and many others)
- **CARL•Connect** (Los Angeles PL)
- **Vega Discover** (Miami-Dade PL)
- **Evergreen** (many state/county systems)
- **Bibliomation/Acorn** (Connecticut libraries)
- **OverDrive/Libby** (digital lending — nearly every US public library)
- **Hoopla** (instant digital access, no holds)

If the catalog blocks automated access, it falls back to generating a search reference table you can use manually.

## Disclosure

Book links to Amazon are affiliate links. As an Amazon Associate, I earn from qualifying purchases. This does not affect the price you pay.

## License

This project is licensed under the [GNU General Public License v3.0](LICENSE). You're free to use, modify, and distribute it — but any derivative work must also be released under GPLv3 and include the same freedoms. See the LICENSE file for full details.
