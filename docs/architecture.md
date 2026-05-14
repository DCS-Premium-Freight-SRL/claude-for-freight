# Architecture

Three layers, the same pattern Anthropic released for legal on 12 May 2026 — adapted
for an industry where the work product is movement, not a document.

```
┌─────────────────────────────────────────────────────────────────┐
│                       PRACTICE PROFILE                          │
│                  CLAUDE.md per plugin per install               │
│   (your company, your lanes, your incoterms, your escalation)   │
└─────────────────────────────────────────────────────────────────┘
                              ▲ reads from
       ┌──────────────────────┼──────────────────────┐
       │                      │                      │
┌──────┴───────┐      ┌───────┴───────┐      ┌───────┴─────────┐
│   PLUGINS    │      │   CONNECTORS  │      │ MANAGED AGENTS  │
│  (skills +   │      │  (MCP servers │      │  (scheduled,    │
│   commands)  │      │   to TMS/rate │      │  off-hours,     │
│              │      │   /customs)   │      │  bounded auth)  │
└──────┬───────┘      └───────┬───────┘      └───────┬─────────┘
       │                      │                      │
       └──────────────────────┼──────────────────────┘
                              ▼
                       Claude (model)
                              │
                              ▼
                    Operator at the desk
                  (or sleeping, for agents)
```

## Why three layers and not one

A monolithic "freight AI" — one prompt, one big system, do everything — fails for
the same reason a monolithic legal AI failed: the work is too varied and the
incentives at each layer are too different.

**Practice profiles** belong to the operator. They contain the moat — the agents
you trust, the lanes you know, the rates you've negotiated. They never leave
the operator's machine. Anyone can fork the repo; nobody else gets your profile.

**Plugins** belong to the industry. They encode practice that's the same across
forwarders — IATA volumetric divisor 6000, CMR has these 24 boxes, the EU customs
value formula adds insurance and freight to the CIF border. This is what's worth
open-sourcing because every forwarder needs the same thing and nobody benefits
from each one rebuilding it.

**Connectors** belong to the vendor ecosystem. They're the bridge between the
plugin (which is generic) and the operator's actual stack (which is specific).
A `cargo.one` connector reads from cargo.one. A `cargo-wise` connector reads
from CargoWise. The plugin doesn't care which one is connected; it asks for a
rate and gets one.

**Managed agents** are the asymmetric piece. The vast majority of plugin runs
are operator-triggered — somebody types a command or asks a question. Managed
agents are the runs that happen *without* a trigger — overnight RFQ monitoring,
weekend AOG response, charter market scanning at 04:00 when somebody might want
to know an aircraft just became available. They run on the same skills the
plugins use; what differs is when they run and who reviews the output.

## The practice profile is everything

If you're tempted to skip the cold-start interview because "we're a small desk,
the defaults are fine" — don't. The defaults are not fine. The defaults are
wrong on purpose, so that the first time a skill runs it asks you a question
you have to answer.

A `cargo-calculation` skill without a profile assumes IATA divisor 6000, EU
trailer 2.4m, kg/cm/m³ units, EUR. That's right for a Bucharest forwarder; it's
wrong for a London forwarder who quotes in GBP, ships transatlantic on courier
divisor 5000, and uses lbs/inches for their US correspondents. Five wrong
defaults compound into a quote nobody will accept.

A `transport-documents` skill without a profile produces a CMR with placeholder
shipper details and "DCS Premium Freight SRL" hardcoded as the issuing
forwarder. That's worse than no output — it looks finished, so somebody might
send it. The profile is what makes the output yours.

The profile lives at:

```
~/.claude/plugins/config/claude-for-freight/<plugin>/CLAUDE.md
```

It's a Markdown file. Open it. Edit it. Re-run setup if it drifts. The skills
are designed to write back to it — "remember that we now use Maersk as primary
on the FE eastbound" — and to surface contradictions ("the profile says
DAP-only outbound, but you just asked me to quote DDP — should I update?").

## Why MCP instead of plugins-only

The plugins know the procedure. They don't know the data. The data lives in
your TMS, your inbox, your customs platform, your rate sheets.

Anthropic's approach — and ours — is to let the model read the data through
the Model Context Protocol rather than forcing every plugin to ship its own
API client for every vendor. Three concrete benefits:

1. **Authentication once.** The operator authorizes the MCP connector once.
   Every plugin that needs that data uses the same connection. No more "this
   plugin wants Gmail access" four times in a row.

2. **Vendor-neutral plugin code.** The `aog-response` skill calls a generic
   "get current rates for OTP–FRA, next 12 hours, AOG priority." Whichever
   rate connector is installed (cargo.one, WebCargo, internal API) returns the
   rates. The skill doesn't care which.

3. **Connector code stays separate.** Vendor-specific authentication, retry
   logic, rate-limit handling, error parsing — all of that lives in the
   connector, not the plugin. A vendor changes their API; one connector
   updates; every plugin keeps working.

## When to use a managed agent vs a plugin command

The rule of thumb:

- **Plugin command** — the operator is at the desk, the answer is needed now,
  the operator will review the output before acting.
- **Managed agent** — the trigger is environmental (an inbox message arrives,
  a milestone is crossed, a market price moves, the clock hits a certain time),
  the operator is not at the desk, the agent needs an Authority Matrix
  ([`authority-matrix.md`](authority-matrix.md)) to decide what to do on its own
  and what to escalate.

Both layers run the same skills. A managed agent that responds to AOG inbox
messages uses exactly the same `aog-response/skills/triage` that an operator
would invoke at the desk. The difference is the wrapper:

- The agent reads the inbox, classifies the message, runs triage.
- If the result fits inside the Authority Matrix's autonomous bounds (e.g.,
  air freight rate request, under €1K, standard lane, business hours), it
  drafts a response and sends.
- If anything falls outside (charter request, DG, over the threshold, unknown
  shipper), it drafts a response, holds it, and notifies the duty operator
  via Telegram/Slack with the approval request.

The skill doesn't change. The wrapper does.

## What this is not

It is not a TMS. It does not store shipments, generate invoices, or hold
master data. Those belong in your operations system.

It is not a substitute for a licensed customs broker, a DGSA, or a forwarding
license. Outputs are drafts for qualified review.

It is not the AI that decides for you. The Authority Matrix is where you
decide what the AI is allowed to decide. Defaults are conservative on purpose;
loosen them after you've seen the skill perform on your actual workflow.

---

For the layer-by-layer build guide, see:

- [`authority-matrix.md`](authority-matrix.md) — the approval gates pattern
- The `freight-forwarding/` plugin — the worked example for layer 1
- The `connectors/` folder — the worked examples for layer 2
- `freight-forwarding/agents/` — the worked examples for layer 3
