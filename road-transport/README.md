# road-transport

*EU groupage / FTL — CMR generation, e-Transport RO declarations, cabotage rules, mobility-package compliance, parking and tachograph rules.*

> **Status: v0.x — roadmap stub.** Skills listed here are planned; some are
> in development at DCS Premium Freight, others are open for community
> contribution. See [`../CONTRIBUTING.md`](../CONTRIBUTING.md) for how to
> claim one.

## Why this plugin exists

Road freight inside the EU is where regulation has moved fastest in 2024–2026: e-Transport RO mandatory expansions, Mobility Package phase-in, posting-of-drivers directive, ETIAS for non-EU drivers. This plugin tracks compliance per shipment.

## Planned skills

- **`cmr-deep`** — full 24-box CMR with carrier reservations and consecutive-carrier logic
- **`e-transport-ro`** — Romania declaration generation and update lifecycle
- **`cabotage-check`** — EU cabotage rules per shipment chain
- **`mobility-package`** — A1 form, posting declaration, return-home requirement
- **`ftl-vs-groupage`** — route economics: dedicated truck vs groupage break-even

## Planned agents

- **`transit-watcher`** — track active e-Transport declarations against expiry

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
