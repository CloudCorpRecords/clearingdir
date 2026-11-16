# Changelog

## 0.1.0 (2026-10-28)

- First release: a forwarding client for the hosted payer-API directory MCP server, a published directory of medical clearinghouse / payer-API vendors. All 4 captured tools are answered locally for tools/list, and every tools/call is forwarded upstream over a streamable-HTTP JSON-RPC session (initialize, notifications/initialized, tools/call, honouring the Mcp-Session-Id header when the upstream issues one).
- Tools: `list_clearinghouses`, `get_clearinghouse`, `vendors_by_function`, `recommend_clearinghouse`. Full surface, no omissions. The capture carries no `title` keys, so none are served; `list_clearinghouses` carries an empty `properties` object and no `required` key in the capture and is served verbatim, and `recommend_clearinghouse` likewise carries no `required` key.
- Upstream results pass through verbatim, including the isError flag; upstream failures surface as `clearingdir error:` text results.
- One bin serves the CLI (one subcommand per tool, plus `--version`) and the stdio MCP server (`mcp` subcommand). The upstream endpoint can be overridden via `PAYERAPIDIRECTORY_MCP_URL`. The upstream record declares no headers, so no auth header is ever sent.
