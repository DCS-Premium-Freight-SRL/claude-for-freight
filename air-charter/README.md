# air-charter

*Full charter / part charter / on-board courier seat — broker comms, GHA dispute resolution, post-flight cost recovery, charter market scanning.*

> **Status: v0.x — roadmap stub.** Skills listed here are planned; some are
> in development at DCS Premium Freight, others are open for community
> contribution. See [`../CONTRIBUTING.md`](../CONTRIBUTING.md) for how to
> claim one.

## Why this plugin exists

Charter is where the largest single losses happen in freight forwarding. A misquoted aircraft type, an unrecovered GHA cost, a missed billing window — €10K to €50K per incident. This plugin encodes the workflows for charter-side risk management.

## Planned skills

- **`charter-pricing`** — aircraft type / payload / lane / repositioning + market reference
- **`broker-comms`** — RFQ to charter brokers (Chapman Freeborn, ACS, Hunt & Palmer style)
- **`gha-dispute`** — ground handling agent post-flight cost dispute (THC, storage, terminal)
- **`post-flight-recovery`** — close the cost loop within carrier invoicing windows

## Planned agents

- **`charter-market-monitor`** — track aircraft availability on key lanes, repositioning opportunities
- **`post-flight-billing-watcher`** — chase backup invoices before they age past customer's invoicing window

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
