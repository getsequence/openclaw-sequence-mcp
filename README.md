# Sequence for OpenClaw

Official OpenClaw plugin for the [Sequence](https://getsequence.io) MCP server — business banking,
treasury and money-movement automation for AI agents.

This is a **declarative bundle**: it ships no code. It registers Sequence's hosted MCP server and a
skill that teaches the agent how to use it. All tools come from the live server, so the tool surface
updates without a plugin release.

## Install

```bash
openclaw plugins install clawhub:@getsequence/sequence
openclaw gateway restart
```

The first Sequence request in chat opens a browser to sign in to your Sequence account (OAuth 2.1
with dynamic client registration and PKCE). Tokens are cached locally under `~/.mcp-auth`. There is
no separate login step.

> `openclaw mcp login sequence` does **not** work here, and that is expected — that command only
> reads servers you added yourself to `mcp.servers` in `openclaw.json`, not servers contributed by a
> plugin. Authentication is handled by the bridge instead.

### Requirements

- Node.js and `npx` available on the machine running the OpenClaw **gateway** (not your chat client).
  The bundle connects through `mcp-remote`, which runs as a local process.
- A Sequence account. Not provisioned yet? <https://app.getsequence.io/agentic/signup>

## What you get

~35 tools, namespaced `sequence__*` — accounts and balances, transactions, transfers, automation
rules, cards, audit log, and guided setup flows.

**Money movement always requires human approval.** A transfer or rule trigger returns
`APPROVAL_PENDING` with a link to approve in the Sequence app; nothing moves until you do, and
unapproved requests auto-deny after 24 hours. Tools also accept `simulation: true` for dry runs.

The tool list is filtered by your role — a viewer sees read-only tools.

## Troubleshooting

| Symptom | Cause |
|---|---|
| No `sequence__*` tools | Gateway not restarted after install, or the plugin is disabled. Check `openclaw plugins inspect sequence --json` and confirm `diagnostics` is empty. |
| No tools even when connected | Account not provisioned — see the signup link above. |
| Tools hidden in a sandboxed setup | Add `bundle-mcp` (or `sequence__*`) to `tools.sandbox.tools.alsoAllow`. |
| All bundle MCP tools missing | `tools.deny` may include `bundle-mcp`. |
| `SSE stream disconnected` in logs | Cosmetic. The server is stateless and has no push channel; request/response is unaffected. |

To override any of the connection settings, add your own `mcp.servers.sequence` entry in
`openclaw.json` — your config always wins over the plugin's default, and `enabled: false` disables it.

## Why `mcp-remote` rather than a direct URL

Sequence's MCP server is remote (streamable HTTP + OAuth), but OpenClaw currently loads **only stdio**
MCP servers from plugin bundles — a bundle declaring `url` + `transport` is installed and then dropped
at load with `bundle MCP servers use unsupported transports or incomplete configs (stdio only today)`.
Verified on `2026.7.1-2` and `2026.9.1-beta.1`.

So the bundle bridges through [`mcp-remote`](https://www.npmjs.com/package/mcp-remote) over stdio,
which also runs the OAuth flow itself. The version is pinned deliberately rather than floating.

## Maintainers: values coupled to the API

These fields must match the production server. The authoritative source is the live Server Card —
one request checks all of them:

```bash
curl -s https://app.getsequence.io/api/mcp/server-card | jq
```

| Bundle field (`.mcp.json`, `openclaw.plugin.json`) | Must match |
|---|---|
| `mcpServers.sequence.args[2]` (URL) | `remotes[0].url` |
| `name` | `title` |
| `description` | `description` |
| `icon` | `icons[0].src` |
| server key `sequence` | `SERVER_NAMES.production` in the app's `mcpConfig.ts` |

`version` is the **bundle's** version and is independent of the server's.

There is deliberately no `openclaw.compat` / `openclaw.build` block in `package.json`: those are for
*code* plugins, and declaring them without an `extensions` entrypoint makes the validator treat this
code-less bundle as a broken code plugin. Last verified against OpenClaw `2026.7.1-2` (and the
stdio-only gate re-confirmed on `2026.9.1-beta.1`).

Do not copy the server's `instructions` into `SKILL.md` — the server already returns them on every
`initialize`, so the agent receives them live. `SKILL.md` stays thin on purpose: it exists for
discovery/triggering and stable domain framing, which is what keeps this repo quiet.

### Publishing

```bash
clawhub package validate .
clawhub package publish . --dry-run
clawhub package publish . --owner getsequence --wait \
  --categories tools,web --topics "banking,treasury,transfers,automation,fintech"
```

Publish from a clean GitHub checkout — ClawHub reads the source repo and commit from it, which is what
earns the `source-linked` verification tier.
