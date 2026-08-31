# Sequence for OpenClaw

**Give your agent access to your business finances — safely.**

[Sequence](https://getsequence.io) is a financial automation platform for businesses. This is the
official OpenClaw plugin: it connects your agent to your Sequence account so it can answer questions
about your money, set up automations, and propose transfers that you approve.

Money never moves without your explicit approval. See [How money movement works](#how-money-movement-works).

---

## What you can ask

Once installed, these all work in plain language:

**Understanding your money**
> "What's my total cash across all accounts?"
> "How much did I spend on contractors last month?"
> "Show me every transfer over $5,000 this quarter."

**Automating it**
> "Set aside 30% of every incoming payment for taxes."
> "What automation rules do I have running?"
> "Pause the payroll rule until next month."

**Moving it**
> "Simulate moving $10k from checking into my tax pod."
> "Move $2,500 to the Emergency Fund." → *you get an approval link; nothing moves until you click it*

Your agent sees your real accounts, balances, transactions, rules and cards — scoped to whatever
your Sequence role already allows.

---

## Requirements

- **A Sequence account.** Don't have one? [Get started](https://app.getsequence.io/agentic/signup).
- **Node.js** on the machine running your OpenClaw gateway. Sign-in runs as a local process there.

## Install

```bash
openclaw plugins install clawhub:@getsequence/sequence --accept-capabilities
openclaw gateway restart
```

The plugin connects to your Sequence account, so OpenClaw asks you to approve that access before
installing — `--accept-capabilities` confirms it up front. Leave the flag off to review the prompt
yourself first.

### Signing in

Just ask your agent a Sequence question. A browser window opens, you sign in to Sequence once, and
that's it — your agent picks up where it left off. Nothing to copy, no API keys to manage, and no
separate login command to run.

Your credentials stay on your own machine.

---

## How money movement works

This is the part worth reading carefully.

**Reading is immediate. Moving money is not.** When your agent proposes a transfer, Sequence returns
a pending approval and a link. Nothing happens until a human opens that link and approves it in the
Sequence app. Unapproved requests expire on their own after 24 hours.

That means:

- An agent cannot move your money on its own — by design, not by configuration.
- You see the amount, source and destination before anything happens.
- Your agent should tell you a transfer is *pending* and hand you the link. If it ever claims money
  has moved without you approving something, treat that as a bug and
  [tell us](https://github.com/getsequence/openclaw-sequence-mcp/issues).

**Previewing is free.** Ask your agent to *simulate* a transfer and it runs a full dry run — same
validation, same result shape, no approval needed and no money involved. Good for checking a rule
before you trust it.

**Your role still applies.** The plugin doesn't widen your permissions. If your Sequence account is
read-only, your agent is read-only too.

---

## What's included

| Area | Examples |
|---|---|
| Accounts & balances | pods, income sources, connected bank accounts, cards |
| Money movement | transfers between your accounts, with approval |
| Automation | create, edit, pause and trigger rules |
| History | transactions, transfers, card activity, audit log |
| Setup | connecting accounts, adding people, issuing cards |

Tools appear in your agent as `sequence__*`. They come from Sequence directly, so new capabilities
show up without updating this plugin.

---

## Troubleshooting

| Symptom | Fix |
|---|---|
| `requires capability consent` when installing | Re-run with `--accept-capabilities`, or install interactively and approve. |
| No Sequence tools in your agent | Restart the gateway: `openclaw gateway restart`. |
| Tools appear but every request fails | Your account may not be set up for agent access yet — [check here](https://app.getsequence.io/agentic/signup). |
| `openclaw mcp login sequence` says no such server | Expected — you don't need it. Sign-in happens automatically on your first Sequence request. |
| Tools missing in a sandboxed setup | Allow `bundle-mcp` in `tools.sandbox.tools.alsoAllow`. |
| `SSE stream disconnected` in the logs | Harmless, safe to ignore. |

Still stuck? [Open an issue](https://github.com/getsequence/openclaw-sequence-mcp/issues) or ask your
agent to contact Sequence support for you.

## Custom setup

Prefer to configure the connection yourself — a different endpoint, your own headers, or an API key
instead of browser sign-in? Add an `mcp.servers.sequence` entry to your `openclaw.json`; your own
config always takes precedence over this plugin's defaults. Set `enabled: false` there to turn the
connection off without uninstalling.

---

## Links

- [Sequence](https://getsequence.io) · [Web app](https://app.getsequence.io)
- [Report an issue](https://github.com/getsequence/openclaw-sequence-mcp/issues)

MIT licensed. This plugin contains no application code — it connects OpenClaw to Sequence's hosted
service and ships guidance for your agent.
