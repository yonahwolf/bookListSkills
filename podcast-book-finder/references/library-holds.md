# Library availability and holds

## Discovering a hold MCP server (during setup)

Look for an MCP server that can search the user's catalog and place holds. Check in this order and stop when you find a good candidate:

1. **Already connected.** List the tools you have now. Any tool whose name or description mentions the library, its catalog vendor, or its digital platform (for example "hold", "checkout", "catalog", "bibliocommons", "overdrive", "libby", "polaris") is a candidate.
2. **The agent's connector directory.** If your environment can search an MCP or connector registry (Claude has a connector and MCP registry search; some Codex and ChatGPT setups have an apps or connectors directory), search it for the library's name, its catalog vendor and its digital platforms.
3. **The web.** Search for `"<catalog vendor> MCP server"`, `"<digital platform> MCP"`, `"<library name> MCP"`, and the same terms on GitHub, the official MCP registry (registry.modelcontextprotocol.io), and community lists such as mcpservers.org and awesome-mcp-servers.
4. **The library itself.** Some libraries publish an API or an official integration; check the library's developer or "apps" page.

For each candidate, note: what it can do (search, place hold, cancel hold, list holds), which formats it covers (print vs. Libby/OverDrive etc.), how it signs in, who maintains it, how recently it was updated, and its license.

Present the best one or two to the user with these caveats stated plainly:

- A third-party server would act on their library account. They should review its code and only install it if they trust it.
- **Do not install or connect anything yourself.** Give the install command or steps and let the user run them. Credentials go into the server's own configuration (environment variables or its sign-in flow), never into the chat or `config.json`.

Once the user has connected it, verify it: run one catalog search through it for a well-known title and confirm the results look like the user's library. Do **not** place a test hold. Then record `hold_mcp` in the config (server name, the search and hold tool names, formats covered, verified date). If nothing suitable exists, set `hold_mcp` to `null`; runs will use hold links instead.

Re-check for a server when the user asks, or when the configured one stops working (then tell the user rather than silently switching).

## Checking availability (each run)

For each book, use the first method that works:

1. **Hold MCP server**, if configured: search by ISBN, falling back to title and author.
2. **The catalog directly**: fetch the catalog's search URL for the title and author. Many catalogs block automated requests; on a 403, CAPTCHA or empty JavaScript shell, stop using this method for the rest of the run.
   - **BiblioCommons** catalogs answer a public JSON search that includes copy and hold counts: `https://gateway.bibliocommons.com/v2/libraries/<subdomain>/bibs/search?query=<title author>&searchType=smart&f_FORMAT=BK` (drop `f_FORMAT` to include ebooks and audiobooks). Read `entities.bibs.*.briefInfo` for the title and `availability` for `status`, `availableCopies`, `totalCopies` and `heldCopies`. A record with 0 total copies is usually digital-only. Link the user to `https://<subdomain>.bibliocommons.com/v2/record/<id>`.
   - If a title isn't found, retry once with a different query (title only, or title and full author name) before marking it ❌.
3. **Web search** scoped to the catalog: `site:<catalog domain> <author last name> <short title>`.
4. **Digital platforms**: search each configured platform (for Libby/OverDrive, `<library>.overdrive.com/search?query=...`).
5. **Fallback**: mark ❓ and add a row to the search reference table with a ready-made catalog search link.

Check every format listed in `preferred_formats`; physical and digital availability often differ.

## Placing holds

Only through a configured, verified `hold_mcp`. Never sign in to a library website, fill in a hold form in a browser, or handle card numbers or PINs.

Candidates for a hold are books with no copy available now in a preferred format, that are not already on hold (check the account's hold list if the server can, and `history.json`).

- **`hold_mode: "ask"`** (interactive runs): list the candidates with format and current wait-list length if known, and ask which to place. Place only the ones the user approves, one call per book, and report the result of each.
- **`hold_mode: "auto"`** (scheduled runs): place holds for up to `max_auto_holds_per_run` candidates, preferring books matching `topics`, then the most recent episodes. List the remaining candidates under **Hold these?**.
- **`hold_mode: "off"`**, or no server: never place holds; include the catalog link for each book so the user can place it themselves.

Record the outcome in `history.json` (`placed`, `declined`, `failed`) so the same book is not offered again. If the server reports a hold limit reached, stop placing holds for the run and say so in the list.
