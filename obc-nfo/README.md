# obc-nfo

*On-Board Courier / Next Flight Out — passport pool management, visa risk scoring, route ladder optimization, courier-of-record paperwork.*

> **Status: v0.x — roadmap stub.** Skills listed here are planned; some are
> in development at DCS Premium Freight, others are open for community
> contribution. See [`../CONTRIBUTING.md`](../CONTRIBUTING.md) for how to
> claim one.

## Why this plugin exists

OBC has its own operational vocabulary: passport pool availability, courier visa coverage, transit visa risk for the specific airport pair, allowable carry vs checked. This plugin handles those decisions in a way the general freight-forwarding plugin can't.

## Planned skills

- **`passport-pool-check`** — which courier in your network can fly the route with valid visas tomorrow
- **`visa-risk-scoring`** — transit visa requirements per nationality per airport pair
- **`courier-routing-ladder`** — fastest commercial flight ladder for the available passports
- **`courier-paperwork`** — Customs Power of Attorney + courier-of-record letter + cash advance docs

## Planned agents

- **`obc-availability-watcher`** — daily pool refresh, expiring visa alerts, courier illness/availability

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
