# cold-chain

*Pharma / perishables — GDP-compliant routing, IATA CEIV-Pharma carrier filter, temperature excursion playbook, qualified packaging selection.*

> **Status: v0.x — roadmap stub.** Skills listed here are planned; some are
> in development at DCS Premium Freight, others are open for community
> contribution. See [`../CONTRIBUTING.md`](../CONTRIBUTING.md) for how to
> claim one.

## Why this plugin exists

Cold chain has narrow tolerance: 2–8°C / 15–25°C / −20°C / dry ice / liquid nitrogen — and excursion outside the window is a customer-side claim event. This plugin encodes the workflows that prevent excursions and the playbooks for when they happen anyway.

## Planned skills

- **`qualified-carrier-filter`** — CEIV-Pharma certification check per lane + carrier
- **`packaging-selection`** — passive vs active container, pre-conditioning window, hold time
- **`temperature-excursion-playbook`** — first-30-min response, customer notification, claim package
- **`gdp-documentation`** — Good Distribution Practice transport record requirements

## Planned agents

- **`excursion-watcher`** — temperature logger pull from IoT-enabled containers, threshold alerts

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
