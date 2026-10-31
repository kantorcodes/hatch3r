# Migration — MCP 2026-07-28 (Current Protocol)

The [official MCP versioning page](https://modelcontextprotocol.io/specification/versioning) now identifies `2026-07-28` as the **current** protocol revision (checked 2026-10-30). `2025-11-25` is an older revision that remains supported by some clients and servers. hatch3r still pins `2025-11-25` in `MCP_DEFAULT_PROTOCOL_VERSION`, but this is its **legacy compatibility default**, not the latest MCP specification. That constant is an advisory marker in generated Claude Code config; it does not by itself negotiate or select the protocol on the wire. Cursor and VS Code Copilot do not receive a top-level `protocolVersion` marker. Confirm actual client/server compatibility before changing the default.

## Re-verification log

`MCP_DEFAULT_PROTOCOL_VERSION` (`src/adapters/mcp-utils.ts`) pins hatch3r's **advisory compatibility default**, which can differ from the current MCP revision. The versioning page describes the protocol's status, not any specific client's tested support. Updating the constant should be a separate change after compatibility tests; this documentation correction does not change runtime behavior.

| Re-checked | Protocol revision marked Current | hatch3r default constant | 2026-07-28 current? | Action taken |
|------------|--------------------------------|-----------------|-----------------------|--------------|
| 2026-07-12 (Cycle-12 Wave-4, D2-SA2.4-16) | 2025-11-25 | 2025-11-25 (match) | No | Historical check; no change at that time |
| 2026-10-30 | 2026-07-28 | 2025-11-25 (legacy default) | Yes | Correct documentation; compatibility change remains untested |

The 2026-07-12 row records the historical status reported then. The [live versioning page](https://modelcontextprotocol.io/specification/versioning) lists `2026-07-28` as **Current** as of 2026-10-30.

## Current hatch3r behavior

- Claude Code `.mcp.json` declares the advisory `protocolVersion: "2025-11-25"` by default (constant `MCP_DEFAULT_PROTOCOL_VERSION` in `src/adapters/mcp-utils.ts`). This marker does not control the transport's negotiated protocol revision. Other adapters do not include this top-level marker.
- Override via `.hatch3r/hatch.json`:
  ```json
  { "mcp": { "servers": ["github"], "protocolVersion": "2026-07-28" } }
  ```
  Set the advisory marker to `2026-07-28` only after verifying that your client and selected servers support that revision; setting a marker alone cannot upgrade a client or server.

## Breaking changes in 2026-07-28

| Change | 2025-11-25 (older revision) | 2026-07-28 (current) | Operator action |
|--------|----------------------|-----------------|-----------------|
| Handshake | `initialize` / `initialized` exchange required | Removed; protocol version + client info travel in `_meta` on every request | None at hatch3r layer; server/client libraries must upgrade |
| Sessions | `Mcp-Session-Id` header, protocol-level sessions | Removed; stateless core, works behind standard load balancers | Assess legacy transport/session dependencies before adopting the current revision |
| Tasks | (n/a) | Lifecycle moved to an extension: `tools/call` returns task handles; clients drive via `tasks/get` / `tasks/update` / `tasks/cancel` | Adopt the Tasks extension only if long-running tools are in use |
| Missing-resource error code | `-32002` (MCP-custom) | `-32602` (JSON-RPC standard Invalid Params) | Update error-code handling in custom server/client glue |
| Roots / Sampling / Logging | Supported | Formally deprecated, twelve-month removal window | Plan replacement before the removal window closes |

## When to pin `protocolVersion: "2026-07-28"`

1. Confirm every MCP server referenced in `mcp.servers` advertises 2026-07-28 support.
2. Confirm your client (Claude Code / Cursor / VS Code Copilot) and selected servers support the current revision and its per-request version handling.
3. Update error-handling glue for the `-32002` → `-32602` change.
4. Set `mcp.protocolVersion` in `.hatch3r/hatch.json` and run `npx hatch3r sync`.

Until those four hold, leave hatch3r's advisory marker at its legacy default. Do not confuse that value with the current MCP specification.

## References

- MCP versioning (current status) — https://modelcontextprotocol.io/specification/versioning (checked 2026-10-30)
- MCP 2026-07-28 specification — https://modelcontextprotocol.io/specification/2026-07-28/
- Historical 2026-07-28 RC announcement — https://blog.modelcontextprotocol.io/posts/2026-07-28-release-candidate/ (accessed 2026-05-27, trust tier: official-docs)
- MCP 2026 roadmap — https://blog.modelcontextprotocol.io/posts/2026-mcp-roadmap/ (accessed 2026-05-27, trust tier: official-docs)
