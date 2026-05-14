---
name: cargo-calculation
description: >
  Compute chargeable weight, CBM, freight ton, and LDM for any shipment given
  dimensions and gross weight. Surfaces the dominant factor (weight vs volume)
  and warns when the shipment crosses a mode-economic threshold. Reads the
  IATA volumetric divisor and trailer dimensions from the practice profile.
  TRIGGER: chargeable weight, volumetric weight, CBM, m³, freight ton, LDM,
  loading meters, dimensional weight, cargo calc, calculează marfa, calculează
  chargeable, cât iese pe air, cât iese pe sea, cât iese pe road.
  Also: "cât iese chargeable pe X colete", "fă calculul pe marfa asta",
  "what's the chargeable on", "how many CBM is", "convert this to LDM".
metadata:
  author: DCS Premium Freight SRL
  version: 1.0.0
  pattern: B  # Prompt + deterministic formulas
---

# Cargo calculation

Deterministic cargo arithmetic. Input: dimensions, weights, mode. Output:
the four canonical metrics with the dominant factor flagged.

## Read the profile first

Before computing anything, check the practice profile for:

- **IATA volumetric divisor** (default 6000; some courier operations use 5000)
- **Units** (default cm/kg; some operations use inch/lb)
- **Trailer base for LDM** (default 2.4m × 13.6m EU semi-trailer)

If the profile is missing, prompt the operator once for the divisor and units,
note in the response that defaults were used, and offer to write the answer
into the profile.

## Inputs to collect

Ask only for what's missing. If the operator says "4 pallets 120×80×140 cm,
250 kg each", you have everything you need; don't ask follow-ups.

Required:
- Number of pieces
- Dimensions per piece — L × W × H, in the unit from profile
- Weight per piece **or** total gross weight
- Mode (air / sea / road) — ask if not stated

Optional (improves output if present):
- Stackable — if yes, halves LDM in the optimistic case (note the assumption)
- Pallet type (EUR / industrial / non-standard) — affects LDM precision
- Commodity — surfaces whether the calculation might trigger a tariff class

## Formulas

### Volumetric weight (air)

```
volumetric_kg = (L_cm × W_cm × H_cm × pieces) / divisor
```

`divisor` from profile (default 6000). Result rounded **up** to the next 0.5 kg
per IATA convention.

### Chargeable weight (air)

```
chargeable_kg = max(gross_kg_total, volumetric_kg)
```

### CBM (sea / road)

```
cbm = (L_cm × W_cm × H_cm × pieces) / 1,000,000
```

### Freight ton (sea — LCL break-bulk convention)

```
freight_ton = max(cbm, gross_tonnes)
```

Where `gross_tonnes = gross_kg_total / 1000`. The carrier bills on whichever
is greater (1 W/M — weight or measurement).

### LDM (road — loading meters)

For non-stackable cargo on a standard trailer:

```
ldm = (L_cm × W_cm × pieces) / (trailer_width_cm × 100)
```

With profile default `trailer_width_cm = 240`:

```
ldm = (L_cm × W_cm × pieces) / 24,000
```

For stackable cargo where double-stacking fits the trailer height:

```
ldm_stacked = ldm / 2
```

Flag the assumption explicitly. *"Assuming double-stackable; LDM is X if
not stackable."*

### Pallet count (informational)

If the dimensions suggest EUR pallets (≤120 × 80 cm footprint):

```
eur_pallets = ceil(footprint_area / (120 × 80))
```

If industrial-pallet footprint (≤120 × 100):

```
ind_pallets = ceil(footprint_area / (120 × 100))
```

Use this as a sanity check, not as the primary output.

## Dominant factor logic

After computing both `chargeable_kg` (air) and `freight_ton` (sea):

- If `volumetric_kg > gross_kg_total` → **volume-dominant** (cargo is light
  for its size; air freight will charge for the box, not the weight)
- Else → **weight-dominant**

For sea:

- If `cbm > gross_tonnes` → **volume-dominant**
- Else → **weight-dominant**

Surface this in the output. Operators use it to decide which carrier-specific
break point matters.

## Threshold warnings

Surface, but don't block, when:

| Crossing | Warning |
|---|---|
| `chargeable_kg > 1,000 kg` air | "Above 1000 kg — General Cargo Rate often beats Minimum/Normal; ask carrier for GCR." |
| `chargeable_kg > 3,000 kg` air | "Above 3 t — consider part charter quote for comparison." |
| `chargeable_kg > 5,000 kg` air | "Above 5 t — full charter range; FCL sea / charter often beats scheduled." |
| `cbm > 15 m³` LCL | "Above 15 CBM — break-even with FCL 20' is usually 13–18 CBM depending on lane; price both." |
| `cbm > 30 m³` LCL | "Above 30 CBM — almost always cheaper as FCL 20'." |
| `ldm > 13.6` road | "Exceeds a single semi-trailer; needs two trucks or mega-trailer (13.8m) — surface to operator." |

These thresholds are conventions, not hard rules. They're worth surfacing
because operators sometimes anchor on the first quote and miss a cheaper mode.

## Output format

Use this exact layout — operators scan it visually.

```
CARGO SUMMARY
─────────────────────────────────────────
Pieces:          [N] × [L × W × H cm]
Gross Weight:    [X] kg
CBM:             [X.XX] m³
─────────────────────────────────────────
AIR  → Volumetric:   [X] kg  (divisor [6000])
     → Chargeable:   [X] kg
SEA  → Freight Ton:  [X.XX] (W/M)
ROAD → LDM:          [X.XX] ldm  (non-stack)
                     [X.XX] ldm  (if stackable)
─────────────────────────────────────────
Dominant factor:     [WEIGHT / VOLUME]
─────────────────────────────────────────
Notes:
[Any threshold warnings]
[Any assumption flags]
```

When mode is specified by the operator, lead with that mode's metric and
collapse the others into a single line:

```
AIR — Chargeable Weight: 720 kg (volumetric 720, gross 540, divisor 6000)
Volume-dominant; consider GCR if quoting >1000 kg lanes.

Other modes for reference: CBM 4.32 m³ · LDM 1.6 (non-stack) · FT 4.32 W/M
```

## Bilingual output

If the operator types in Romanian, respond in Romanian. The layout stays the
same; labels translate:

| EN | RO |
|---|---|
| Pieces | Colete |
| Gross Weight | Greutate brută |
| CBM | CBM (sau metri cubi) |
| Volumetric | Volumetric |
| Chargeable | Chargeable (sau taxabil) |
| Freight Ton | Freight Ton (W/M) |
| Loading Meters | Metri liniari (LDM) |
| Dominant factor | Factor dominant |
| Weight / Volume | Greutate / Volum |
| Notes | Observații |

## Common errors to refuse

- **Negative dimensions or weights** — reject the input, ask for correction.
- **Zero pieces** — reject, ask for correction.
- **Unit ambiguity** ("60 × 40 × 30" with no unit) — ask once before computing.
  Don't guess.
- **Mixing units in one shipment** ("3 pallets 120 cm × 80 cm × 100 cm and
  2 pallets in inches") — calculate each group separately, sum at the end,
  surface the conversion.

## Edge cases

**Single piece, partial dimensions** — if the operator gives only L × W (e.g.,
a flat crate), ask for H. Without H you cannot compute volumetric.

**Total weight without per-piece breakdown** — compute as if uniform; flag
that the calculation assumes uniform weight distribution. Real-world this
matters for LDM stacking decisions.

**Non-standard pallets** (oversized industrial, half-EUR, custom crates) —
fall back to physical L × W × H of each unit; do not assume pallet footprint.

**Volume given directly as CBM** — fine, use it. Skip the dimensional
calculation. Note in output that CBM was input, not derived.

## When the operator asks for the carrier-side perspective

If the operator says *"and what would the carrier charge — give me a rate"* —
this skill does **not** quote rates. Hand off to the operator's rate
platform connector if installed, or refuse cleanly:

> "I don't have buy rates loaded. The chargeable is [X]. Try cargo.one /
> WebCargo / your rate sheet for the [origin–destination] lane to convert
> to a quote."
