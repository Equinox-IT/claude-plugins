# Approval record — eq-m365-files

| | |
|---|---|
| Plugin | `eq-m365-files@equinox` |
| Source | First-party, in this repository: `plugins/eq-m365-files` |
| Gateway source | Private Equinox repository. Its design, controls and risk assessment are kept there, not here |
| Version | 0.1.0 |
| Reviewed | Not yet. Drafted 13 November 2026 |
| Outcome | **Draft — do not merge.** Blocked on the items under "Before this can merge" |

This is the scoped alternative to the native Microsoft 365 connector that the integration
register (INT-STD-001) calls for. The plugin is only the pointer: it tells Claude Code where the
gateway is and how to sign in. All access control is in the gateway, not in the plugin.

## Capability surface

| Surface | Present | Detail |
|---|---|---|
| Hooks | none | No `hooks/` directory, no `hooks` key |
| MCP servers | 1 | `eq-m365-files`: remote HTTP server at `https://m365-gateway.equinox.co.nz/files`. Nothing runs locally |
| Subagents | none | |
| Slash commands | none | |
| Skills | none | |
| Bundled code | none | Two JSON files: `plugin.json` and `.mcp.json` |
| Outbound network | yes | The gateway's internal address, and Microsoft sign-in (`login.microsoftonline.com`) from the person's browser |
| Writes outside project | no | Claude Code stores the sign-in token in its own credential store |

## What the server gives Claude

Five read-only tools: `list_locations`, `browse`, `find`, `get_metadata`, `read_document`.
There is no write, send, share or delete tool. People sign in as themselves, and access is
decided by the gateway, not by this plugin. How the gateway decides, and what it records, is
documented with the gateway.

## Findings

**Clear — nothing executes on the endpoint.** The plugin carries no hooks, commands, skills or
code. Removing the entry removes the server.

**Clear — no secret in the plugin.** The sign-in client is a public client. The `clientId` is an
identifier, not a credential, and there is no client secret anywhere in the design.

**Watch — the names are part of permission rules.** A server that comes from a plugin is
registered as `plugin:eq-m365-files:eq-m365-files`, and its tools are named
`mcp__plugin_eq-m365-files_eq-m365-files__<tool>`, for example
`mcp__plugin_eq-m365-files_eq-m365-files__read_document`. That differs from the pilot, where the
server was added by hand and the tools are `mcp__eq-m365-files__<tool>`. A permission rule written
for the pilot form never matches the plugin form. Saved permissions and any rule in
`claude-policy` must use the plugin form, and changing either name later means touching every managed
profile. (Claude Code documentation, "Plugin-provided MCP servers".)

## Before this can merge

1. **A production gateway exists.** No production gateway is running yet, so merging now would
   give every user a connector that cannot connect.
2. **The real client id.** `.mcp.json` carries a placeholder (`00000000-…`). Replace it with the
   production Claude Code client registration's id.
3. **Sign-in from a plugin is proven.** The pilot registers the server with `claude mcp add-json`.
   The `oauth` settings (pre-registered client, fixed callback port, pinned scopes) are documented
   for servers added that way and in `.mcp.json`. The documentation for plugin-provided servers
   says nothing about them either way, so this must be tested on one machine. If it does not work,
   the fallback is a managed server entry instead of a plugin.
4. **Managed policy allows it by exact URL.** `claude-policy` must allow
   `https://m365-gateway.equinox.co.nz/files` by `serverUrl` and nothing broader. The documentation
   says the allow and deny lists are checked for servers a plugin adds, and that a server name is
   not a security control. Two things to check in `claude-policy` first: if a `managed-mcp.json`
   is deployed it takes exclusive control and a plugin's server will not load at all; and once any
   `serverUrl` entry exists, every remote server must match one.
5. **Checklist gates 05 to 07** run against the production gateway: data and egress (recorded
   with the gateway), the named owner, and pilot and rollback on one machine.
6. **Register updated:** a row for this connector in `integration-register.md`, and the demand
   log closed off.
