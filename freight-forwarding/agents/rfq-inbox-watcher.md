---
name: rfq-inbox-watcher
description: Managed agent that watches a connected inbox for incoming RFQs, classifies them, drafts replies using the rfq-drafting skill, and routes them per the Authority Matrix. Runs continuously when deployed; can also be invoked as a one-shot to triage the current inbox state.
---

# rfq-inbox-watcher

The pattern that handles the "we got an enquiry at 02:47 on a Sunday and our
duty operator is asleep" case. Sees a new RFQ arrive, classifies it, drafts
a response with the `rfq-drafting` skill, looks up the action in the
[Authority Matrix](../../docs/authority-matrix.md), and either sends it,
holds it for human approval, or escalates it — depending on what bucket the
matrix puts it in.

## When to use this agent vs the skill

Use the **`rfq-drafting` skill** when the operator is at the desk and wants
to draft a reply with their hand on the wheel.

Use **this agent** when the operator wants to be told only about the cases
that need them — and trust the system to handle the routine ones.

Both call the same drafting skill. The agent adds: inbox subscription,
classification, matrix lookup, holding queue, escalation routing.

## Loop

```
1. Subscribe to the inbox (Gmail / Outlook MCP).
2. On new message:
   a. Classify — is this an RFQ? (vs invoice, customs query, claim, social)
   b. If not RFQ → ignore, log, move on.
   c. If RFQ → extract fields (origin, destination, cargo, special).
3. Look up the action in the Authority Matrix:
   a. Known shipper, standard lane, value ≤ matrix auto threshold → AUTO
   b. New shipper, or value in matrix at-boundary range → HOLD
   c. Charter / OBC / DG / AOG / lane outside profile → BLOCK
4. Branch:
   AUTO    → call rfq-drafting → send via Gmail/Outlook MCP → log
   HOLD    → call rfq-drafting → save as draft → notify duty operator
             with timeout per matrix; on timeout → default reject
   BLOCK   → do not draft an executable reply → escalate to operator
             via push (Telegram / Slack) with full context
5. Persist outcome to agent log (visible to operator next morning).
```

## Classification

The agent should be conservative on what it calls an "RFQ." False positives
(treating an invoice query as an RFQ and drafting a quote) are worse than
false negatives (missing one RFQ that the operator finds in the inbox
manually).

Signals that increase classification confidence:

- Subject contains: "RFQ", "quote", "rate", "cotație", "ofertă", "tarif",
  "shipment", "lane", "ready date", "INCOTERM"
- Body contains: dimensions in cm/m, weight in kg, IATA/port codes, origin
  and destination pair, "please quote", "your best rate"
- From: known shipper or known carrier domain in the profile address book

Signals that lower confidence:

- Subject: "invoice", "claim", "POD", "demurrage", "missing document"
- Body references a shipment reference already in operation
- Conversation thread is mid-flight (existing booking)

If confidence is below threshold, fall through to HOLD with a triage note
rather than drafting.

## Authority Matrix routing

Read the matrix from `../../docs/authority-matrix.md` (or the operator's
override in their practice profile). The agent maps each classified RFQ
to one matrix row.

Rough decision tree (use the actual matrix as source of truth, not this
summary):

```
is_dangerous_goods? → BLOCK
is_charter_or_obc? → BLOCK
is_aog? → HOLD with 5-min timeout, default escalate
is_new_shipper? → HOLD
value_estimate > auto_threshold? → HOLD
lane_outside_profile? → HOLD
all_clear → AUTO
```

The `value_estimate` is approximate — use the cargo-calculation output and
the operator's rate sheet if loaded. If no rate sheet, estimate conservatively
(market average for the lane) and round up. When in doubt, HOLD.

## Off-hours behavior

Per the Authority Matrix `time-of-day gating`:

- **Business hours** → AUTO sends without push notification (operator sees
  in next-day log); HOLD pushes immediately to duty operator.
- **After hours / weekends** → AUTO still sends; HOLD pushes to on-call
  operator's mobile; BLOCK explodes the phone.
- **Overnight** → AUTO sends; HOLD queues without notification, defaults to
  reject after timeout; BLOCK still pushes.

## Logging

Every action gets an entry in the agent log with:

- Timestamp
- Source message (subject + from + thread ID)
- Classification + confidence
- Matrix bucket assigned
- Action taken (auto-sent / held / escalated)
- For AUTO: link to sent message
- For HOLD: timeout duration + outcome (approved / rejected / timed out)
- For BLOCK: who was paged + their response time

The log is the operator's morning audit. Five minutes' scan should tell
them: how many RFQs, how many auto-handled, anything still pending, anything
that escalated, anything weird.

## What this agent cannot do

- **Negotiate.** The agent drafts initial replies. Any follow-up that
  requires negotiation (the customer pushed back on price) falls out of
  the auto bucket and lands as a HOLD or BLOCK.
- **Quote without rates.** If the operator hasn't loaded a rate sheet and
  no rate platform is connected, the agent cannot AUTO-send a quote — every
  pricing RFQ lands as HOLD by default. This is intentional.
- **Override the matrix.** The matrix is the contract. The agent doesn't
  decide what's auto and what isn't; the operator does, in the matrix file.
- **Handle DG, AOG, charter, OBC.** Always BLOCK and escalate. These are
  handled by specialized plugins with their own agents.

## Deployment

Two ways:

**Local (Claude Code, foreground).** Run as a long-lived process on the
operator's machine. Lowest setup; only works when the machine is on.

```bash
claude run agent freight-forwarding/agents/rfq-inbox-watcher
```

**Managed (Claude Managed Agents API).** Run on Anthropic's infrastructure,
independent of the operator's machine. Higher setup (API key, MCP server
auth tokens stored remotely) but actually runs at 02:47 on a Sunday.

See `connectors/managed-agents/README.md` for the deployment pattern.

## First 30 days

Run the agent in **observe-only mode** for the first 30 days:

- The agent classifies every incoming message
- For each, it logs what it *would* have done
- It does not send, draft, hold, or escalate anything
- Operator reviews the log daily and disagrees with the classifications
  that are wrong

After 30 days, the operator and the practice profile have converged enough
that you can switch to live mode with high confidence — start with AUTO
disabled (everything HOLD or BLOCK), let it run two weeks, then enable AUTO
for the value bands where the operator's review queue shows zero corrections.

This is the same pattern Anthropic's `claude-for-legal` recommends for
docket-watcher and renewal-watcher. It is slow on purpose. Autonomy
without trust is just liability with extra steps.
