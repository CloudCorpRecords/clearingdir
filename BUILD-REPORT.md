# BUILD REPORT — clearingdir

Clean-room rebuild of `com.dropwatchhq/payer-api-directory`
v1.0.0 (remote-only MCP server at
`https://clearinghouse.dropwatchhq.com/mcp`), built
2026-10-28 as a forwarding client. Package:
`@reneturcios/clearingdir` 0.1.0, registry name
`io.github.CloudCorpRecords/clearingdir`.

## Surface built

Full surface: **all 4 captured tools built**, no omissions.

| Tool | Required fields as served |
|------|---------------------------|
| `list_clearinghouses` | NO `required` key in the capture — served verbatim, none invented; its inputSchema carries an EMPTY `properties` object, served verbatim |
| `get_clearinghouse` | `slug` |
| `vendors_by_function` | `function` |
| `recommend_clearinghouse` | NO `required` key in the capture — served verbatim, none invented |

Each served tool carries exactly the keys `name`, `description`,
`inputSchema`, exactly as captured: the capture carries NO `title`
keys, so none are served. The tool literals were generated
programmatically from
`harness/fixtures/payerdirectory-tools-original.json` by the script
at `/tmp/b47-clearingdir-scratch/gen_forward.py` (parse +
re-serialise into `src/forward.ts`) — never retyped — and a
post-generation check (the generator's own round-trip assertion,
the registration-integrity test suite, plus the parity harness)
confirmed the served literals deep-equal the fixture's 4 tool
objects, in order. Tools are registered via the MCP SDK low-level
Server API (`ListToolsRequestSchema` / `CallToolRequestSchema`
handlers), not McpServer+zod.

## Fixture conflicts

**None.** The spec and the fixture agree on all 4 tools' names,
descriptions, required fields (including the two tools with no
`required` key: `list_clearinghouses`,
`recommend_clearinghouse`), the empty `properties` object on
`list_clearinghouses`, and every property shape. The spec's note
that the capture carries no `title` keys matches the fixture
exactly.

## Labelled reconstructions (from the spec's Wire section)

- Upstream endpoint pinned: `https://clearinghouse.dropwatchhq.com/mcp`.
- Env override `PAYERAPIDIRECTORY_MCP_URL` — labelled reconstruction.
- The registry record declares NO headers on the remote: no auth
  header is ever sent. Tests assert that no `Authorization` header
  is sent under any environment — including with key-like env vars
  (`PAYERAPIDIRECTORY_API_KEY`, `API_KEY`, `AUTHORIZATION`) set —
  and that no `X-API-Key` header is ever sent; upstream requests
  carry only `Content-Type`, `Accept` and, when issued,
  `Mcp-Session-Id`.
- Error rendering `clearingdir error:` text for upstream failures —
  labelled reconstruction.
- Upstream results pass through verbatim, including `isError` and
  extra fields. tools/call is forwarded as streamable-HTTP
  JSON-RPC: initialize (protocolVersion `2025-03-26`) →
  notifications/initialized → tools/call, honouring
  `Mcp-Session-Id` when issued; plain-JSON and SSE replies are
  both parsed. The forwarding core takes an injectable fetch;
  tests stub the upstream — no live network calls in tests.
- Directory judgment (from the spec, recorded by the coordinator
  on the firsthand capture read): the whole surface is a published
  vendor DIRECTORY. The three reads return published directory
  records (access model, API style, X12 functions, dated +
  confidence-tagged pricing signals — published/reported/none).
  `recommend_clearinghouse` transparently ranks vendors for a
  practice from caller-supplied NEEDS (functions needed,
  dev-integration and budget flags) and returns a scored shortlist
  with the reasoning for each: a transparent ranking over
  published catalogue data about companies, driven by the
  caller's own requirements — no caller-named subject is screened,
  no exclusion list is checked, nothing is gated (the awardsignal
  market-intelligence distinction). No personal data (the
  directory is about vendor companies), no payment, metering,
  contact, signup or send tool exists in the surface.

## Deviations

None.

## Lockfile

- Route: generated clean from birth in the build dir with no
  `node_modules` and no pre-existing lockfile:
  `npm install --package-lock-only --ignore-scripts --offline`
  (Route 3).
- Parses as JSON; `name` == `@reneturcios/clearingdir`.
- Package entries: **128**.
- sha256: `1501d43d0a33e72429beb2dfc31808ed1530216169751d3bd349d57e9b598ef7`
  (re-verified unchanged after the subsequent offline `npm ci`).

## server.json

- Description exactly: "Medical clearinghouse and payer-API
  vendor directory: records, X12 functions, rankings." —
  **87 characters**.
- npm package entry carries `"version": "0.1.0"`; the OCI entry
  (`ghcr.io/cloudcorprecords/clearingdir:0.1.0`) carries **no**
  `version` field. Both entries carry packageArguments
  `[{type: positional, value: mcp}]` and transport stdio.
  Dockerfile (node:20-alpine, both stages) carries the label
  `io.modelcontextprotocol.server.name="io.github.CloudCorpRecords/clearingdir"`.
  Container workflow `IMAGE` is `ghcr.io/cloudcorprecords/clearingdir`
  and its smoke test asserts the server name `clearingdir`.

## Verification (all run 2026-10-28)

1. `npx tsc --noEmit` → exit 0.
2. `npm run build` → exit 0; real `dist/` produced (cli, forward,
   index, mcp, types — .js, .d.ts, .js.map each).
3. `node dist/cli.js --version` → `0.1.0`.
4. `npm test` → **55 tests, 55 pass, 0 fail** (registration-integrity
   suite deep-equals the served tools/list against the fixture's 4
   tools, including the no-title assertions, the empty-`properties`
   assertion on `list_clearinghouses`, and the no-required
   assertions, plus session handling/reuse, env override, the
   no-Authorization-header assertions under any environment,
   verbatim argument forwarding and result pass-through including
   upstream isError, SSE parsing, error rendering, MCP-layer
   validation, and CLI behaviour).
5. Parity: `python3 /home/hatch/workspace/mcp-rebuild-100/harness/parity.py --original /home/hatch/workspace/mcp-rebuild-100/harness/fixtures/payerdirectory-tools-original.json --rebuild "stdio:node /home/hatch/workspace/mcp-rebuild-100/builds/clearingdir/dist/cli.js mcp" --out /home/hatch/workspace/mcp-rebuild-100/builds/clearingdir/parity-out` → **verdict PASS, parity 100.0**
   (4/4 identical, 0 missing, 0 extra, no schema diffs). Output in
   `parity-out/`.
