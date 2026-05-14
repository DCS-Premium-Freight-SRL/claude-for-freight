---
name: customs-checklist
description: Direct invocation of the customs-checklist skill. Use for EU/RO import or export pre-clearance checklist, TARIC duty lookup, VAT estimation, and regulated-goods documentation flags.
---

# /freight-forwarding:customs-checklist

Triggers the [`customs-checklist`](../skills/customs-checklist/SKILL.md) skill
directly.

## Usage

```
/freight-forwarding:customs-checklist
```

Provide:

- Direction (import / export / transit)
- Origin and destination
- Commodity description (or HS code if you have it)
- Invoice value and currency
- Incoterms
- Mode (air / sea / road / rail)

## Examples

> Import checklist for EVA slippers from China, HS 6402.99, FOB Shanghai
> $48,000, sea LCL to Constanța.

> Export OTP → Cairo, auto spare parts, EXW invoice value €18,400,
> ready Friday — what do I need?

> TARIC and VAT estimation on lithium-ion power banks from Shenzhen,
> declared value €15,000, air freight.

## What you get back

- Baseline required documents
- Conditional documents (CE, REACH, RoHS, phytosanitary, etc.) flagged by
  commodity
- RO-specific items (e-Transport, e-Factura post-clearance)
- Indicative customs value, duty, excise, VAT base, VAT
- Total estimated clearance cost

## Disclaimer

Indicative only. Classification, valuation, and filing are the licensed
declarant's responsibility under EU Customs Code Art. 18 / RO Law 86/2006.
For binding classification, apply for a BTI.
