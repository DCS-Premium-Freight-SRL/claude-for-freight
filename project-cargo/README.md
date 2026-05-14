# project-cargo

*Heavy lift / OOG / break-bulk projects — method statement, route survey, escort coordination, multi-modal handover, lift plan.*

> **Status: v0.x — roadmap stub.** Skills listed here are planned; some are
> in development at DCS Premium Freight, others are open for community
> contribution. See [`../CONTRIBUTING.md`](../CONTRIBUTING.md) for how to
> claim one.

## Why this plugin exists

Project cargo is bespoke per shipment. The workflow is method statement → route survey → permit and escort → lift plan → handover, repeated. This plugin encodes the structure so each project doesn't restart from a blank page.

## Planned skills

- **`method-statement`** — drafting per cargo type, route, equipment
- **`route-survey`** — overpass clearances, road class limits, escort requirements
- **`permit-coordination`** — abnormal load permits per country in route
- **`lift-plan`** — crane spec, ground bearing, rigging selection
- **`modal-handover`** — interface points between road / sea / barge / rail in one project

## Planned agents

- **`permit-expiry-tracker`** — across active project cargo permits

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
