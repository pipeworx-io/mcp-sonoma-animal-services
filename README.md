# @pipeworx/sonoma-animal-services

Live shelter intake and outcome records from Sonoma County Animal Services
(Santa Rosa / Sonoma County, CA) — recent intakes, animal lookup (including
"is this animal still at the shelter"), and outcome summaries.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1576+ live data sources.

## Tools

- `sonoma_recent_intakes(days?, animal_type?, intake_type?, limit?)` —
  recent intakes (default last 30 days), with a total count for the window.
- `sonoma_search_animal(animal_id?, name?, breed?, in_shelter_only?, limit?)`
  — look up an animal by id, name, or breed. `in_shelter_only=true` answers
  "is there a [breed] at the shelter right now" — an animal with no
  `outcome_date` yet is still in the shelter.
- `sonoma_outcomes_summary(since?, group_by?)` — counts of outcomes grouped
  by `outcome_type`, `animal_type`, or `month`.

## Auth

Keyless. Pass your own Socrata app token via `_apiKey` for higher rate
limits.

## Data sources

- <https://data.sonomacounty.ca.gov/resource/924a-vesw.json> — "Animal
  Shelter Intake and Outcome": one row per animal, with both intake and
  outcome fields on the same record (unlike Austin, which splits intakes and
  outcomes into separate datasets). `outcome_date IS NULL` means the animal
  has not yet left the shelter.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "sonoma-animal-services": {
      "url": "https://gateway.pipeworx.io/sonoma-animal-services/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/sonoma-animal-services/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 1576+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## Standalone (no gateway account)

This package also runs as a local stdio MCP server — no Pipeworx account, no
gateway round-trip:

```json
{
  "mcpServers": {
    "sonoma-animal-services": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-sonoma-animal-services"]
    }
  }
}
```

Or run it directly to confirm it starts:

```bash
npx -y @pipeworx/mcp-sonoma-animal-services
```

It speaks MCP over stdin/stdout and answers `initialize`/`tools/list`/`tools/call`
for **only** this pack's tools — none of the shared meta-tools the gateway
connection above adds. Same source, same tools, no ask_pipeworx routing.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Sonoma Animal Services data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
