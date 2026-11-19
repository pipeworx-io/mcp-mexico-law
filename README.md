# @pipeworx/mexico-law

Mexican federal laws and codes as published by the Chamber of Deputies: find a
law by words in its name, then read its reform record and its text, whole or
one article at a time, with the in-force or abrogated status the publisher
lists it under.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 2015+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `mx_law_search(query, status?, limit?)` — find laws and codes by Spanish words
  in the name (for example "protección de datos personales", "amparo").
  Searches the publisher's list of laws in force and its list of abrogated
  laws, and says which list each result is on, with the DOF date of the
  original publication and of the last reform (or of the abrogation).
- `mx_law_get(law_id | name, article?, max_chars?)` — one law (`lfpdppp`, `ccf`,
  `cpeum` for the Constitution, or `abro:...` for an abrogated one): list
  status, original and last-reform dates, any note the list carries (for
  example a scheduled abrogation), the reform decrees with their DOF dates and
  the text. `article` (`"7"`, `"1916 Bis"`) returns one article, flagged when it
  begins "Se deroga". Unknown ids return `found:false` with a reason.

## Auth

Keyless.

## Data sources

- Mexico, Cámara de Diputados (Secretaría General, Secretaría de Servicios
  Parlamentarios), Leyes Federales — the lists of laws in force and abrogated,
  each law's reform decrees and its official text.

Status is the list the publisher keeps the law on (`vigente` or `abrogada`),
plus any remark the list carries; nothing is derived from the reform record.
Coverage is federal laws and codes plus the Constitution; regulations
(reglamentos), state laws and the Diario Oficial itself are not included.
Text is read from the publisher's PDF, so tables are flattened and running page
headers are removed; amended text carries the publisher's own "reformado DOF"
annotations. `data_as_of` is the date of the live read. Nothing is stored; every
call reads the publisher live.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "mexico-law": {
      "url": "https://gateway.pipeworx.io/mexico-law/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/mexico-law/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 2015+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/mx_law_search \
  -H 'Content-Type: application/json' \
  -d '{"query":"datos personales"}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/mx_law_search`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-mexico-law` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/mexico-law/mcp` endpoint shown above. It needs network access;
there is no offline mode.

```json
{
  "mcpServers": {
    "mexico-law": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-mexico-law"]
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
ask_pipeworx({ question: "your question about Mexico Law data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
