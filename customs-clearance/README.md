# customs-clearance

*EU import/export deep-dive — TARIC lookup, DV1 workflow, EUR.1 preparation, AEO simplifications, ICS2 entry summary.*

> **Status: v0.x — roadmap stub.** Skills listed here are planned; some are
> in development at DCS Premium Freight, others are open for community
> contribution. See [`../CONTRIBUTING.md`](../CONTRIBUTING.md) for how to
> claim one.

## Why this plugin exists

The `freight-forwarding` plugin includes a baseline customs checklist. This plugin is the deep-dive — for forwarders running their own customs declarations under simplified or AEO procedures, not just handing off to a broker.

## Planned skills

- **`taric-lookup`** — HS code search with confidence scoring against commodity description
- **`dv1-workflow`** — Customs Value Declaration when value exceeds €20,000
- **`origin-statement`** — EUR.1 / EUR-MED / GSP Form A / origin declaration drafting
- **`aeo-simplifications`** — entry in declarant's records, centralized clearance, deferred VAT account
- **`ics2-ens`** — Entry Summary Declaration data pre-arrival for air / sea / road / rail
- **`duty-suspension`** — IPR / OPR / End-use / Customs Warehouse procedures
- **`cbam-transitional`** — Carbon Border Adjustment Mechanism quarterly reporting

## Planned agents

- **`tariff-change-watcher`** — monitor TARIC and DG-Taxud feeds for HS code rate changes affecting profile commodities
- **`binding-tariff-tracker`** — BTI applications status (when submitted) and validity expiry

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
