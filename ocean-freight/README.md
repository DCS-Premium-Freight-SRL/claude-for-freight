# ocean-freight

*FCL / LCL / breakbulk — booking workflow, B/L draft (original vs sea waybill vs telex release), demurrage tracker, ETA reconciliation, port-of-discharge documentation.*

> **Status: v0.x — roadmap stub.** Skills listed here are planned; some are
> in development at DCS Premium Freight, others are open for community
> contribution. See [`../CONTRIBUTING.md`](../CONTRIBUTING.md) for how to
> claim one.

## Why this plugin exists

Ocean has slower operational rhythm than air but worse cost surprises (demurrage, detention, GRI, peak season surcharges). This plugin tracks the things that bite forwarders 30 days after the cargo arrived.

## Planned skills

- **`fcl-booking`** — container type / 20' vs 40' vs 40HC vs reefer, equipment availability
- **`lcl-consolidation`** — break-bulk vs CFS handling, vol/weight rules, MAFI rolling
- **`bl-type-decision`** — original / sea waybill / telex release / express bill
- **`demurrage-tracker`** — free days per origin + destination + carrier, escalation cost
- **`eta-reconciliation`** — actual vs scheduled ETA, claim windows

## Planned agents

- **`demurrage-watcher`** — daily free-day countdown across active shipments
- **`vessel-tracker`** — schedule changes, port omissions, alternative-vessel suggestion

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
