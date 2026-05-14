# dangerous-goods

*IATA DGR / ADR / IMDG / RID — UN number lookup, packing instruction, segregation matrix, Shipper's Declaration draft, CAO/PAX air check.*

> **Status: v0.x — roadmap stub.** Skills listed here are planned; some are
> in development at DCS Premium Freight, others are open for community
> contribution. See [`../CONTRIBUTING.md`](../CONTRIBUTING.md) for how to
> claim one.

## Why this plugin exists

Dangerous goods is where the regulatory and criminal exposure is. Mis-declaration is a felony in most jurisdictions. This plugin encodes the workflows under qualified DGSA / DG Approval Officer review. It never auto-classifies.

## Planned skills

- **`un-lookup`** — UN number from commodity description, with confidence and synonyms
- **`packing-instruction`** — applicable PI per mode (DGR for air, ADR for road, IMDG for sea)
- **`segregation-check`** — compatibility between multiple DG entries in the same shipment
- **`shippers-declaration`** — DGD draft for review by qualified DG approval
- **`cao-pax-decision`** — Cargo Aircraft Only vs Passenger Aircraft eligibility
- **`li-battery-rules`** — lithium battery specifics (UN 3480 / 3481 / 3090 / 3091) - the most common DG class in freight

## Planned agents

- **`dg-regulation-watcher`** — IATA DGR annual edition changes (Jan 1 each year), ADR biennial changes, restricted commodities feed

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
