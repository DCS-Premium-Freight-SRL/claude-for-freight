# aog-response

*Aircraft on Ground — sub-30-minute response, part triage, charter trigger, recovery quote, post-flight cost reconciliation.*

> **Status: v0.x — roadmap stub.** Skills listed here are planned; some are
> in development at DCS Premium Freight, others are open for community
> contribution. See [`../CONTRIBUTING.md`](../CONTRIBUTING.md) for how to
> claim one.

## Why this plugin exists

AOG is the highest-value, highest-stress workflow in time-critical freight. A wrong call costs five-figure penalties per hour of aircraft downtime. This plugin encodes the workflow: triage incoming AOG, lock the part location, decide charter vs commercial, dispatch, track, reconcile costs after.

## Planned skills

- **`aog-triage`** — classify severity, part type, aircraft type, current location, MTBR pressure
- **`routing-ladder`** — fastest available routing across commercial schedules + charter options
- **`charter-trigger`** — when charter beats commercial: thresholds and decision matrix
- **`post-flight-recovery`** — reconcile actual carrier costs against quoted; chase backup invoices
- **`customer-comms-aog`** — pre-departure / departure / arrival / delivered template ladder

## Planned agents

- **`aog-incident-responder`** — 5-min triage window, hard 5-min Authority Matrix timeout, defaults to escalate
- **`post-flight-cost-watcher`** — chase carrier final invoices, flag delays beyond DHL invoicing window

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
