# @pipeworx/fr-cnil-sanctions

CNIL (Commission nationale de l'informatique et des libertés, France's data-protection authority): the list of sanctions its restricted committee has pronounced against organisations since 2011: administrative fines, injunctions and penalty payments, with the failings found. French text is kept as published.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 2006+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `cnil_sanctions_search(query?, year?, min_fine_eur?, limit?)` -- searches the list, newest first. The keyword matches the organisation type, the themes and the outcome. Each hit: id, date, deliberation reference when the CNIL prints one, organisation type, themes, outcome, `fine_eur` parsed from the outcome, simplified-procedure flag, link to the deliberation. `individuals_dropped` counts the rows not returned.
- `cnil_sanction_get(id)` -- one row by its id (date plus position within that date, such as `2026-01-08-1`). An unknown id is `found:false`.

Every response carries `source` and `data_as_of` (the newest sanction date).

## Auth

Keyless.

## Data sources

- CNIL, sanctions pronounced by the restricted committee. Read live per call, nothing stored.

Privacy: organisations only. The CNIL does not print organisation names in this list, only the type ("opérateur de téléphonie mobile", "société de vente à distance", "ministère", "commune"); the deliberation it links to names the organisation. Rows against individuals (doctors, lawyers, dentists, orthophonists, notaries, private persons, election candidates) are dropped and counted in `individuals_dropped`.

Traps worth knowing: a company name will not match, because the list carries none; search by sector or theme. Ids are positions within a date. `fine_eur` is null when the outcome carries no amount (rappel à l'ordre, injonction alone).

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "fr-cnil-sanctions": {
      "url": "https://gateway.pipeworx.io/fr-cnil-sanctions/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/fr-cnil-sanctions/mcp` returns the tools in the table
above **plus the shared Pipeworx meta-tools** — `ask_pipeworx`,
`discover_tools`, `search_within`, `remember`/`recall` and the rest of the
gateway-wide set. So the tool count you see is larger than this table: a
single-pack endpoint currently lists roughly 30 shared tools alongside the
pack's own. The connection's `initialize` response states its exact scope, and
is the authoritative answer for a given day.

This is deliberate, not multiplexing by accident. The meta-tools are what let a
scoped connection answer a question this pack does not cover — via
`ask_pipeworx`, which routes across the whole catalog — without you adding a
second MCP server. There is currently no way to mount a pack endpoint without
them; if the extra schemas cost you more context than the routing is worth,
connect to the full gateway once rather than to several pack endpoints.

Or connect to the full Pipeworx gateway to get every pack's tools listed
directly, instead of just this one's:

```json
{
  "mcpServers": {
    "pipeworx": {
      "url": "https://gateway.pipeworx.io/mcp"
    }
  }
}
```

Both URLs reach the same gateway and the same 2006+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/cnil_sanctions_search \
  -H 'Content-Type: application/json' \
  -d '{"query":"sécurité des données","year":2026}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/cnil_sanctions_search`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-fr-cnil-sanctions` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/fr-cnil-sanctions/mcp` endpoint shown above. It needs network access;
there is no offline mode.

```json
{
  "mcpServers": {
    "fr-cnil-sanctions": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-fr-cnil-sanctions"]
    }
  }
}
```

Without a key it calls anonymously, capped at 50 calls a day. A free account
raises that to 200 a day: get a key at https://pipeworx.io/signup and add
`"env": { "PIPEWORX_API_KEY": "<your key>" }` to the server entry.
The key is sent as a Bearer token, the same way a hosted connection sends it,
so calls are metered and billed exactly as they are there.

It lists **only** this pack's tools by default. Set `PIPEWORX_META_TOOLS=1` to
also list `ask_pipeworx` and the other gateway-wide tools.

Behind an HTTPS proxy: Node's built-in `fetch` does not read `HTTPS_PROXY` by
default. Newer Node releases honour it when `NODE_USE_ENV_PROXY=1` is set; check
your Node version's documentation.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Fr Cnil Sanctions data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
