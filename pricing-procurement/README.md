# pricing-procurement

*Forwarder-side RFP responses — tender matrix, lane benchmark, win/loss memo, pricing committee decision support.*

> **Status: v0.x — roadmap stub.** Skills listed here are planned; some are
> in development at DCS Premium Freight, others are open for community
> contribution. See [`../CONTRIBUTING.md`](../CONTRIBUTING.md) for how to
> claim one.

## Why this plugin exists

Most forwarder pricing is reactive — quote per RFQ at the desk. Major tenders (annual RFPs from Tier-1 customers, government framework agreements) need structured pricing: lane benchmark, margin floor justification, win/loss tracking against the tender population. This plugin encodes that.

## Planned skills

- **`tender-matrix`** — structured response to a tender across N lanes
- **`lane-benchmark`** — your historical rates + market reference per lane
- **`margin-floor-justification`** — why each lane is priced where it is, for pricing committee review
- **`win-loss-memo`** — post-tender analysis: where we lost, where we left margin on the table

## Planned agents

- **`tender-deadline-tracker`** — across active RFPs, who is doing what by when

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
