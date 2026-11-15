# @pipeworx/ca-recalls

Canadian federal recalls and safety alerts: consumer product, food, health product and vehicle notices from Health Canada, the Canadian Food Inspection Agency and Transport Canada, as published in the Government of Canada's Recalls and Safety Alerts.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1957+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `ca_recalls_search(query?, lang?, date_from?, date_to?, limit?, page?)` -- search by keyword in English or French (`lang`), optionally with a publication date range. With no keyword it lists the newest notices. Returns id, title, notice type and date, plus `data_as_of`, the list searched and the total the publisher reports. One 15-row page per call in the publisher's own relevance order; date filters apply to the fetched page and the response says how many rows they hid.
- `ca_recall_get(id)` -- one notice in full: product, brand, category, the issue and hazard as stated, affected models and descriptions, what to do, number sold in Canada, period sold, place of origin, importer and manufacturer (companies), recall number and issuing authority. Accepts a notice id from search or a recall number such as `RA-82889`. Unknown id returns `found:false`.

## Auth

Keyless.

## Data sources

- Government of Canada, Recalls and Safety Alerts (Health Canada, Canadian Food Inspection Agency, Transport Canada).

Read live per request; nothing is stored. Notices are in English or French as requested.

Traps worth knowing:

- The publisher lists about twenty thousand notices; results are ranked by its own relevance, not by date, so use `date_from` with a keyword and read several pages if you need the newest.
- Company phone numbers and email addresses are deliberately not returned; the notice link has them.
- A French notice has its own id (`avis-rappel/...`); the recall number (`RA-...`) works in both languages.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "ca-recalls": {
      "url": "https://gateway.pipeworx.io/ca-recalls/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/ca-recalls/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 1957+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/ca_recalls_search \
  -H 'Content-Type: application/json' \
  -d '{"query":"power bank","limit":5}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/ca_recalls_search`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-ca-recalls` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/ca-recalls/mcp` endpoint shown above. It needs network access;
there is no offline mode.

```json
{
  "mcpServers": {
    "ca-recalls": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-ca-recalls"]
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
ask_pipeworx({ question: "your question about Ca Recalls data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
