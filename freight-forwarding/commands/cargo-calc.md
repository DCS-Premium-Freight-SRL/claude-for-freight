---
name: cargo-calc
description: Direct invocation of the cargo-calculation skill. Use when you want a deterministic chargeable/CBM/LDM/freight-ton calculation and don't want to wait for the skill to be inferred from natural language.
---

# /freight-forwarding:cargo-calc

Triggers the [`cargo-calculation`](../skills/cargo-calculation/SKILL.md) skill
directly.

## Usage

```
/freight-forwarding:cargo-calc
```

Then provide the cargo details in any natural form:

> 4 pallets 120×80×140 cm, 250 kg each, air freight

Or structured:

> pieces: 4
> dimensions per piece: 120 × 80 × 140 cm
> weight per piece: 250 kg
> mode: air

You can also paste an email containing the cargo specs — the skill extracts.

## What you get back

A summary with chargeable weight, CBM, freight ton, LDM, and the dominant
factor (weight vs volume). See the skill for the full output format.

## Notes

- If the practice profile is missing, you'll be prompted to confirm the IATA
  divisor (default 6000) and units (default cm/kg) once.
- The skill computes; it doesn't quote rates. To go from chargeable weight
  to a quote, you need a rate connector (`cargo.one`, `WebCargo`) or your
  rate sheet.
