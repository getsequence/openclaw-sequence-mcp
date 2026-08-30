---
name: sequence
description: Sequence — business banking, treasury and money-movement automation. Read account balances, transactions and transfers; inspect, create and trigger automation rules; move money between accounts and pods under human approval. Use for questions about company cash, balances, spend, runway, tax set-asides, recurring transfers, and payment automation. Not a general accounting, invoicing, payroll or bookkeeping tool.
metadata: {"openclaw":{"homepage":"https://app.getsequence.io","emoji":"🏦"}}
---

# Sequence

Sequence is a financial automation platform for businesses. This plugin connects to the official
Sequence MCP server, which exposes the user's own accounts and automations.

## First use

The connection authenticates over OAuth on the first Sequence request — a browser opens for sign-in
and the tokens are cached locally. There is no separate login command to run.

If tools are missing entirely, the user is not provisioned: send them to
<https://app.getsequence.io/agentic/signup>.

## Domain model

Read these before interpreting a request; the vocabulary is specific and easy to get wrong.

- **Account** — anything holding or moving money. Types include **pods** (goal-based buckets, e.g.
  "Tax Savings"), **income sources** (where incoming funds land, e.g. "Salary"), connected
  **external accounts** (real bank accounts), and **cards**.
- **Beneficiary** — the legal entity (person or business) that owns an account. **Money can only
  move between accounts owned by the same beneficiary.** A business beneficiary stays unusable until
  its verification (KYB) is approved.
- **Rule** — an automation that moves money when a trigger fires. Rules can be created, edited,
  activated/deactivated, and triggered manually.
- **Transfer** — one individual movement of money.

Balances need two steps: `listAccounts` never returns balances, and `getAccount` is the only way to
read one. `getAllBalances` and `getFinancialSnapshot` explain how to assemble the full picture —
read them rather than guessing.

## Every call takes a `reason`

All tools accept (and often require) a `reason`: a short description of what the user is trying to
accomplish. It is used for attribution. Fill it in meaningfully.

## Moving money — read this before any write

**`APPROVAL_PENDING` is the normal, successful outcome, not an error.** A non-simulated
`createTransfer` or `triggerRule` returns `status: APPROVAL_PENDING` plus an `approvalUrl`, because
money movement over this connection always requires a human to approve in the Sequence app.

- **Surface the `approvalUrl`** to the user and tell them the action is proposed, pending approval.
- **Never say the money moved.** Nothing has moved.
- **Never retry.** A retry without the original idempotency key creates a *second* pending approval
  for the same intent. If it looks like it failed, check `listTransfers` before acting.
- Unapproved requests auto-deny after 24 hours.

**Use `simulation: true` to preview.** Simulated calls are dry runs, are never approval-gated, and
return `executionMode: "SIMULATION"`. Prefer simulating first for anything non-obvious, and say
clearly that the result was a simulation.

Confirm the amount, source and destination with the user before proposing any transfer. Amounts are
in **cents** (`amountInCents`), minimum 100.

## Reading results

- IDs are full UUIDs and are needed for follow-up calls — never truncate or abbreviate them.
- Account and routing numbers are masked to the last 4 digits. The full numbers are never available;
  do not imply you have them.
- If a tool returns an error, surface the full message and code — they carry specific remediation.
- A tool you expected may be absent: the tool list is filtered by the signed-in user's role, so a
  viewer sees read-only tools. That is intended, not a fault.
