# automotive-logistics

*OEM and Tier-1 supplier workflows — Stellantis MMOG/LE, line-stop response, milk-run optimization, MAS-Loop coordination, AS9100/IATF16949 documentation.*

> **Status: v0.x — roadmap stub.** Skills listed here are planned; some are
> in development at DCS Premium Freight, others are open for community
> contribution. See [`../CONTRIBUTING.md`](../CONTRIBUTING.md) for how to
> claim one.

## Why this plugin exists

Automotive is operationally different from general freight: line-stop windows are measured in minutes, not days; documentation must match OEM-specific EDI standards; carrier choice is constrained by quality system certification. This plugin encodes that.

## Planned skills

- **`line-stop-triage`** — classify severity, expected downtime cost, escalation ladder
- **`mas-loop`** — Manufacturer Approved Supplier loop transport coordination
- **`milk-run-design`** — multi-pickup consolidation with time-window optimization
- **`edi-translation`** — VDA / Odette / ANSI X12 message mapping to internal shipment data
- **`quality-record`** — IATF 16949 / AS9100 transport record audit trail

## Planned agents

- **`line-stop-responder`** — sub-15-min response window on inbound line-stop alerts

## Dependencies

This plugin layers on top of [`../freight-forwarding/`](../freight-forwarding/).
Install the base plugin first; this one adds the specialized workflow on top.

The base plugin's `cargo-calculation`, `rfq-drafting`, and `transport-documents`
skills are reused. This plugin's skills override or supplement only where the
workflow genuinely differs from the general case.

## How to contribute

The bar is a single working skill — one workflow, end-to-end, with the
practice-profile integration. A complete plugin doesn't need to ship all
the skills above on day one. Open an issue describing which skill you want
to build, agree on the scope, and submit a PR.

See [`../CONTRIBUTING.md`](../CONTRIBUTING.md).
