# Authority Matrix

The pattern that lets a freight desk run autonomously while you sleep without
making the kind of mistake that costs €40K to undo.

This is the layer that turns "we have an AI that drafts replies" into "we
have an AI that handles the inbox overnight." The difference is not the
model, the prompts, or the skills. The difference is whether the system
knows when to act, when to pause, and when to wake somebody up.

## The principle

Every action a managed agent can take has three possible outcomes:

1. **Inside autonomous bounds** — act, log the action, move on.
2. **At the boundary** — draft the action, hold it, notify a human, expire
   after a timeout, default to "do not act."
3. **Outside autonomous bounds — absolute block** — do not draft, do not
   hold, escalate immediately, refuse to act even if a human says yes via
   a fast channel (Telegram approve button). Human must take over in their
   primary tool.

The matrix is what defines, for your operation, which actions fall into
which bucket. The defaults below are the ones DCS Premium Freight runs in
production. Yours should look different — that's the point.

## Default matrix (DCS reference)

| Decision | Auto | At-boundary | Block |
|---|---|---|---|
| **Quote response to known shipper, standard lane, ≤€1K** | ✓ | | |
| **Quote response to known shipper, standard lane, €1K–€5K** | | ✓ (15-min approval, default reject) | |
| **Quote response to known shipper, standard lane, >€5K** | | | ✓ |
| **Quote response to new shipper, any value** | | ✓ (15-min) | |
| **Charter request (full or part)** | | | ✓ |
| **OBC / NFO request** | | | ✓ |
| **Dangerous goods shipment** | | | ✓ |
| **Lane outside operator's profile** | | ✓ (operator confirms before any reply) | |
| **Customer credit hold flag set** | | | ✓ |
| **AOG flag in subject or body** | | ✓ (5-min approval, default escalate) | |
| **Booking confirmation issuance** | | ✓ (always — never auto) | |
| **CMR / AWB / HBL final issuance** | | | ✓ (always — never auto) |
| **Invoice draft** | ✓ | | |
| **Invoice send to customer** | | | ✓ |
| **Email reply: rate-card-lookup answerable, internal partner** | ✓ | | |
| **Email reply: rate-card-lookup answerable, customer** | | ✓ (15-min) | |
| **Email reply: requires negotiation** | | | ✓ |

## Time-of-day gating

The same decision can be in different buckets depending on when it happens.

- **Business hours (08:00–18:00, Monday–Friday)** — defaults above apply.
- **After hours (18:00–22:00 weekdays, 08:00–14:00 weekends)** — at-boundary
  approvals route to the on-call operator's mobile (Telegram / Slack push).
  Auto actions still execute.
- **Overnight (22:00–06:00, daily)** — auto actions execute. At-boundary
  approvals queue without push notification, default reject after the timeout.
  Block actions still escalate via push.
- **Weekends excluding on-call window** — auto only, no human pings, everything
  else queues to Monday.

The intent: a managed agent that watches the inbox at 02:47 on a Sunday should
quote a standard €600 air-freight enquiry from a known shipper without waking
anybody, hold a €3K enquiry for Monday morning review, and explode the phone
when an AOG message arrives.

## Timeouts

Every at-boundary action has a timeout. When it expires, the action takes a
default — and the default is conservative.

| Action category | Timeout | Default on expiry |
|---|---|---|
| Routine quote in matrix range | 15 min | Reject (do not send) |
| AOG response | 5 min | Escalate (do not even queue) |
| Operator confirmation on lane-outside-profile | 30 min | Reject |
| Confirmation of automation-suggested rate vs market | 60 min | Reject |

The 5-minute AOG timeout is deliberately tight. AOG response time is the
metric the customer cares about; an automation that holds the reply for 15
minutes while waiting for approval is worse than no automation.

## Block actions never become auto actions

Three categories that never auto-execute regardless of operator preference:

- **Charter** — financial exposure is structurally different; one wrong booking
  is a multi-month profit.
- **Dangerous goods** — regulatory exposure, criminal liability under IATA DGR
  and ADR; never automate the call on whether something can ship as DG.
- **Customer-facing financial commitments (final invoices, credit notes,
  binding quotes above the threshold)** — too easy to send something
  irreversible.

If the operator wants to loosen these — for example, treat sub-€2K part
charter as auto for a specific aircraft type and lane — the place to do it
is in their practice profile, with the loosening logged and re-asked every
30 days. The default stays "block."

## Implementation pattern

The matrix is read by every managed agent before it dispatches an action.
A canonical agent loop looks like this:

```
1. Trigger fires (inbox message, milestone crossed, clock event).
2. Agent classifies the trigger (what kind of decision is needed).
3. Agent runs the relevant skill to produce a draft action.
4. Agent looks up the action in the Authority Matrix.
5. Branch:
   - auto       → execute, log
   - boundary   → hold, notify human, await response or timeout
   - block      → escalate, refuse to draft an executable action
```

Step 4 is where the matrix lives. It's a Markdown table the operator can edit;
the agent code reads it at start-up. There is no separate JSON schema, no API,
no admin UI. If you want to change what's auto and what's at-boundary, you
edit the table.

## What this gets right that other autonomy approaches don't

**It's legible.** The operator can read the matrix and know exactly what the
system will and won't do. No "the AI decided" — every action that fires
autonomously was on a line in a table somebody approved.

**It's reviewable.** Every auto action is logged. Every at-boundary action
that timed out and rejected by default is logged. The operator can scan a
week of agent activity in under five minutes.

**It's tunable without code.** Loosen a row, tighten a row, add a row — the
file is Markdown. No deploy, no config service.

**It defaults to inaction.** The most common failure mode of autonomous
systems is acting when they shouldn't. The matrix bias is "if in doubt,
do not act." Skills that produce drafts always have a human between the draft
and the world unless explicitly listed as auto.

## What it doesn't solve

The matrix tells the agent what's allowed. It doesn't tell the agent how to
handle the situation well. If the underlying skill drafts a bad quote, the
matrix happily auto-sends a bad quote that happens to fall in the auto
bracket.

The matrix is the second line of defence. The skill quality is the first.
A practice profile with one good RFQ example does more for skill quality
than tightening the matrix from €1K to €500.

## See also

- [`architecture.md`](architecture.md) — where managed agents fit in the
  three-layer model
- The `freight-forwarding/agents/` folder — worked examples of agents that
  consult this matrix
