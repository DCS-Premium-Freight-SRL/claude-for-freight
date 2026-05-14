# Claude for Freight

A suite of plugins for freight forwarding workflows — air, sea, road, rail, and time-critical.

> Maintained by **DCS Premium Freight SRL** (Bucharest, Romania) — a neutral freight forwarder
> running these patterns in production every day across 13 DHL Express network contracts,
> automotive logistics for Stellantis, and time-critical operations for retail and pharma clients.

This is the pattern. Fork it, run the cold-start interview against your own playbook,
keep what works, throw out the rest.

---

## What this is

Three layers, the same architecture Anthropic released for legal on 12 May 2026 — adapted
for an industry where the deliverable is a moving aircraft, a sealed container, or an
on-board courier on the next flight out, not a redlined contract.

**Practice-area plugins.** Each plugin packages the tasks a freight desk runs most often —
RFQ drafting, cargo calculation, transport documents, customs checklist, AOG response,
charter pricing, DG declaration. Skills are Markdown, edited by ops staff, not developers.

**MCP connectors.** Bring your operational reality into Claude — TMS (CargoWise, Riege Scope,
StarCargo), e-AWB (IATA Cargo XML, ONE Record), rate platforms (cargo.one, WebCargo),
customs systems (RO ANAF, EU DGTAXUD), and the productivity stack everyone already runs
(Gmail, Drive, Slack, Microsoft 365).

**Managed agents.** The patterns that need to run while no one is watching — RFQ watcher
overnight, AOG incident responder, charter pricing monitor, shipment milestone alerter,
demurrage/detention escalator. Built around an **Authority Matrix** pattern (auto-process
under €1K, autonomous booking 20:00–06:00 and weekends, hard block above €5K or any
charter / dangerous goods / OBC shipment) — see [`docs/authority-matrix.md`](docs/authority-matrix.md).

---

## The plugins

| Plugin | Status | Built for |
|---|---|---|
| [`freight-forwarding`](freight-forwarding/) | **v1.0** ✅ | General multi-modal desk — RFQ, cargo calc, CMR/AWB/HBL/MBL, customs checklist |
| [`customs-clearance`](customs-clearance/) | v0.x 🚧 | EU import/export — TARIC lookup, DV1, EUR.1, AEO workflows |
| [`aog-response`](aog-response/) | v0.x 🚧 | Aircraft on Ground — sub-30-min response, parts triage, charter trigger |
| [`obc-nfo`](obc-nfo/) | v0.x 🚧 | On-Board Courier / Next Flight Out — passport pool, visa risk, route ladder |
| [`air-charter`](air-charter/) | v0.x 🚧 | Full and part charter — broker comms, post-flight cost recovery, GHA dispute |
| [`dangerous-goods`](dangerous-goods/) | v0.x 🚧 | IATA DGR / ADR / IMDG — UN number lookup, packing instruction, DGD draft |
| [`automotive-logistics`](automotive-logistics/) | v0.x 🚧 | OEM/Tier-1 — Stellantis MMOG/LE, line-stop response, milk run, MAS-Loop |
| [`ocean-freight`](ocean-freight/) | v0.x 🚧 | FCL/LCL — booking, B/L draft, demurrage tracker, ETA reconciliation |
| [`road-transport`](road-transport/) | v0.x 🚧 | EU groupage and FTL — CMR, e-Transport RO, cabotage rules |
| [`cold-chain`](cold-chain/) | v0.x 🚧 | Pharma / perishables — GDP, IATA CEIV, temperature excursion playbook |
| [`project-cargo`](project-cargo/) | v0.x 🚧 | Heavy lift / OOG — method statement, route survey, escort coordination |
| [`pricing-procurement`](pricing-procurement/) | v0.x 🚧 | Forwarder-side RFP responses — tender matrix, lane benchmark, win/loss memo |

Every plugin contains a `commands/cold-start-interview.md` that learns your playbook
(your rate cards, your escalation matrix, who signs what, which agents you trust on
which lane) and writes it to a `CLAUDE.md` practice profile. Every skill in the plugin
reads from that profile.

---

## Why this exists

Freight forwarding is what Anthropic's announcement called the second-most fragmented
knowledge-work function we've measured — behind legal, ahead of accounting. A typical
desk operator switches between TMS, carrier portals, customs, Excel rate sheets, Gmail,
WhatsApp, Outlook calendars, and a tariff database in the same hour, copying numbers
across by hand.

The big TMS vendors (CargoWise, Riege, Magaya) won't fix this because their incentive
is to keep the data inside their suite. The carrier portals won't fix this because their
incentive is the opposite. AI assistants without operational context (a free ChatGPT tab)
won't fix this because they don't know what your margin is on the Frankfurt lane or who
your back-up trucker is when the primary cancels.

Claude for Freight fixes it the same way Claude for Legal fixed it for in-house counsel —
by letting one model read from all of those systems through MCP, run the workflow with
a skill that encodes how you actually quote, and hand off to a managed agent when the
work happens at 02:47 on a Sunday morning and there's nobody at the desk.

We built this because we needed it. We're open-sourcing it because the alternative is
twelve separate forwarders all paying twelve separate consultancies to rebuild the same
RFQ-drafting prompt.

---

## Install

You need Claude Code or Claude Cowork. Then:

```bash
git clone https://github.com/dcs-freight/claude-for-freight.git
cd claude-for-freight
```

In Claude Code:

```
/plugin marketplace add /path/to/claude-for-freight
/plugin install freight-forwarding@claude-for-freight
```

Restart Claude Code (this step is not optional — the plugin isn't live until you restart).

Then run the cold-start interview:

```
/freight-forwarding:cold-start-interview
```

Takes 10–15 minutes. It will ask about your company, the lanes you cover, your standard
incoterms, who signs off on charter quotes above what threshold, which agents you have
in which countries. The output is a `CLAUDE.md` practice profile every other skill reads from.

See [`QUICKSTART.md`](QUICKSTART.md) for the longer version including Cowork install,
MCP connector authentication, and the user-scope vs project-scope question.

---

## Disclaimer

Every output from these plugins is a draft for desk-operator review — not a freight
quote you can send to a customer without checking, not a customs declaration you can
file without a licensed broker, not a DG approval. They are built with guardrails that
reflect that: cargo calculations show their formula, customs duties surface the TARIC
code they used, charter quotes flag the assumptions, and the Authority Matrix blocks
any autonomous booking above the threshold you set.

A licensed forwarder, a licensed customs broker, a DGSA — somebody qualified for the
specific decision — reviews and takes professional responsibility for anything that
leaves the building. These plugins make that review faster. They do not replace it.

These plugins do not represent Anthropic's positions on freight or transport law.
DCS Premium Freight SRL maintains them in good faith based on operational practice
under Romanian and EU law; jurisdictions vary and the law in many of these areas
(e-Transport, ICS2, CBAM, PPWR) is unsettled and evolving.

---

## Contributing

See [`CONTRIBUTING.md`](CONTRIBUTING.md). Short version: open an issue describing the
workflow before you write the skill. The bar for merging is "another forwarder could
fork this and run it tomorrow without calling us." Vendor-specific plugins (a plugin
that only works if you're a CargoWise customer) belong as `connectors/`, not as plugins.

---

Licensed under Apache 2.0 — see [`LICENSE`](LICENSE). Use it commercially, modify it,
ship it inside a closed product, keep the copyright notice. That's the deal.

Copyright © 2026 DCS Premium Freight SRL.
