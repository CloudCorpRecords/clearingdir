# clearingdir

Which medical clearinghouses support the X12 functions your practice actually uses, what do they charge, and how hard are they to integrate with? clearingdir is a local forwarding client for the hosted payer-API directory MCP service: it answers `tools/list` locally from the captured tool definitions and forwards every call to the service, returning the answers exactly as the service sends them back.

The directory covers vendor companies only — clearinghouses and payer-API vendors — with their access models, API styles, supported X12 functions, and dated pricing signals tagged by confidence (published, reported, or none). The one judgment-flavoured read, `recommend_clearinghouse`, ranks those published vendors against needs you state yourself (the functions you need, whether developer integration matters, whether budget is tight) and shows the reasoning behind each score: a transparent ranking over published catalogue data, not a screen of any person or company, and nothing is gated by it. This client treats the whole surface that way: it forwards the question and passes the answer through.

All 4 captured tools are built; there are no omissions.

## The tools

- **`list_clearinghouses`** — Every vendor in the directory, each with its access model, API style, supported X12 functions, and whether its pricing is public. Takes no parameters at all — the captured schema carries an empty `properties` object.
- **`get_clearinghouse`** — The full record for one vendor: what it does, dated and confidence-tagged pricing signals, access requirements, and how well it fits a small practice. Requires `slug` (for example `availity`, `stedi`, or `office-ally`).
- **`vendors_by_function`** — Which vendors support a given X12 function. Requires `function`, one of `eligibility`, `claims`, `era`, `claim-status`, or `prior-auth`.
- **`recommend_clearinghouse`** — A transparently ranked shortlist of vendors for a practice, with the reasoning for each score. Optional inputs: `functions` (a list drawn from `eligibility`, `claims`, `era`, `status`, `priorauth`), `devIntegration`, and `budgetSensitive`. No required parameters.

## Install and run

As an MCP server via npm:

```bash
npx -y @reneturcios/clearingdir mcp
```

MCP client configuration:

```json
{
  "mcpServers": {
    "clearingdir": {
      "command": "npx",
      "args": ["-y", "@reneturcios/clearingdir", "mcp"]
    }
  }
}
```

As a container:

```bash
docker run -i ghcr.io/cloudcorprecords/clearingdir
docker run --rm ghcr.io/cloudcorprecords/clearingdir list_clearinghouses '{}'
```

## CLI

One binary serves both the command line and the MCP server. Each tool is a subcommand taking a single JSON string with that tool's arguments:

```bash
clearingdir list_clearinghouses '{}'
clearingdir get_clearinghouse '{"slug":"stedi"}'
clearingdir vendors_by_function '{"function":"eligibility"}'
clearingdir recommend_clearinghouse '{"functions":["claims","era"],"budgetSensitive":true}'
clearingdir --version
```

Exit codes: `0` when the forwarded call succeeds, `1` when the upstream result is an error, and `2` for usage problems such as an unknown subcommand or a malformed JSON argument.

## Configuration

- `PAYERAPIDIRECTORY_MCP_URL` — Overrides the upstream endpoint. The default is the pinned hosted endpoint, `https://clearinghouse.dropwatchhq.com/mcp`. (The override variable is a reconstruction added by this rebuild.)

The upstream registry record declares no headers, so no authentication header is ever sent; the tool surface was captured keyless.

Upstream failures — a JSON-RPC error from the service, a non-2xx response, an unreadable body, or a request that fails outright — are reported to the caller as an error result whose text starts with `clearingdir error:`.

## How forwarding works

The first forwarded call opens a session: an `initialize` request (protocol version `2025-03-26`), followed by a `notifications/initialized` notification, then the `tools/call` itself. If the service answers the initialize request with an `Mcp-Session-Id` header, that value is carried on every later request, and the open session is reused for the rest of the process. Replies are accepted either as plain JSON or as server-sent events. Successful results pass through verbatim, including any `isError` flag and any extra fields the service includes.

## Development

```bash
npm install
npm run build   # tsc -> dist/ (dist/cli.js is the bin)
npm test        # tsx --test test/*.test.ts — stubbed upstream, no network
npm run dev     # run the CLI from source
```

The test suite checks the served tool list against the captured surface (all 4 tools, deep-equal, in order, with no titles invented, the empty `properties` object on `list_clearinghouses` preserved, and no `required` key added to the two tools that carry none), exercises the session handshake and its reuse, verifies that no `Authorization` header is ever sent under any environment, and verifies that arguments go upstream untouched and results come back untouched.

## License

MIT — see `LICENSE`. Copyright (c) 2026 Rene Turcios.
