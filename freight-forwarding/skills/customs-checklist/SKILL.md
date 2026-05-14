---
name: customs-checklist
description: >
  Pre-clearance checklist for import and export into the EU (RO-default).
  Lists required documents, surfaces likely TARIC duty rate from commodity
  description or HS code, estimates customs value and VAT base, flags
  regulated goods that need extra paperwork (CE, REACH, RoHS, phytosanitary,
  veterinary, dual-use, CITES). Does not file — outputs a checklist for
  the licensed declarant.
  TRIGGER: customs checklist, declarație vamală, customs declaration, TARIC,
  HS code lookup, cod vamal, duty rate, taxa vamală, TVA import, import VAT,
  customs documents, regim vamal, vămuire, AEO, EORI, ATR, EUR.1, ICS2,
  e-Transport, T1, T2, NCTS, transit declaration.
  Also: "ce documente trebuie pt vamă", "what's the duty on", "cât e taxa
  pe", "verifică ce trebuie pentru import", "checklist vamă", "TARIC pentru
  HS", "fă-mi lista de documente pentru clearance".
metadata:
  author: DCS Premium Freight SRL
  version: 1.0.0
  pattern: A
---

# Customs checklist

Pre-clearance checklist for EU import and export. Romanian operations are
the default (RO ANAF, DGT-RO TARIC implementation, e-Transport requirements);
other EU member states adapt easily — flag the country in the practice
profile or in the operator's request.

This skill **does not file declarations**. It produces a structured checklist
the licensed customs declarant uses as input. The final classification, the
final value, and the actual filing are the declarant's responsibility under
EU Customs Code (UCC) Article 18 / RO Law 86/2006.

## Read the profile first

From `CLAUDE.md`:

- Country of operation (default RO if profile is RO-based)
- AEO status (Authorized Economic Operator) — changes some thresholds
- Whether the operator holds a customs warehouse, simplified procedures, etc.
- Preferred customs broker for handover
- e-Transport user code if Romania

## Direction

First classify: **import** (entering EU customs territory) or **export**
(leaving), or **transit** (T1 / T2 through EU).

If ambiguous, ask. The required documents differ substantially.

## Import workflow

### Required documents (baseline)

For any import into the EU:

- [ ] **Commercial Invoice** — `valoare în vamă` / customs value supporting
- [ ] **Packing List** — pieces, weights, dimensions
- [ ] **Transport document** — AWB / MBL / HBL / CMR / CIM as applicable
- [ ] **Customs Value Declaration (DV1)** if value > **EUR 20,000**
- [ ] **EORI number** of the importer
- [ ] **Power of Attorney** to the customs broker if filing on importer's behalf

### Conditional documents (by commodity / origin)

Surface these when the commodity description or HS code suggests they apply.

| Trigger | Document |
|---|---|
| Preferential origin claimed | EUR.1 / EUR-MED / GSP Form A / Origin Declaration on invoice |
| Origin China + value > €6,000 + electrical/electronic | CE Declaration of Conformity (DoC) |
| Cosmetics, toys, electronics for retail | CE marking + technical file + EU representative |
| Chemicals | REACH registration / SVHC declaration / Safety Data Sheet |
| Electrical/electronic | RoHS DoC + WEEE registration in destination |
| Food / feed | Health certificate + CHED-D (BIP entry) |
| Plants / wood / wood packaging | Phytosanitary certificate + ISPM 15 mark on wood |
| Animal products | Veterinary certificate + CHED-A/P entry |
| Endangered species / certain leathers | CITES permit |
| Pharmaceuticals | GDP-compliant transport + import permit + Qualified Person release |
| Dual-use items | Export licence (origin side) + end-use statement |
| Radioactive | ADR class 7 + specific permits |
| Excise goods (alcohol, tobacco, fuel) | e-AD + excise number |

### Romania-specific additions

- [ ] **e-Transport** declaration before truck crosses RO border (mandatory
  since 2022, expanded 2025–2026 — check current scope)
- [ ] **SAFER / RO e-Factura** triggers if invoicing inside RO post-clearance
- [ ] **EORI-RO** validation for shipper if first import

### Customs value formula

```
Customs Value = CIF value at EU border
              = Cost of goods (commercial invoice)
              + International freight (to EU border)
              + Insurance (to EU border)
              + Any other adjustments per UCC Art. 71 (commissions,
                royalties, assists)
              − Deductions per UCC Art. 72 (post-importation costs
                if separately invoiced and identifiable)
```

If the operator gives EXW invoice value + transport invoice, compute the
adjustment. If incoterm is already CIF or CIP at EU border, the invoice
value approximates the customs value (still verify any commissions/royalties).

### Duty and VAT estimation

For a given **HS code** (8-digit TARIC for RO):

1. Look up the duty rate from TARIC if known, or surface as "TARIC rate to be
   confirmed by declarant" if not certain.
2. Compute:

```
Customs Duty = Customs Value × TARIC duty rate
Excise Duty  = (commodity-dependent — surface only if commodity is excisable)
VAT Base     = Customs Value + Customs Duty + Excise
VAT          = VAT Base × VAT rate
```

**VAT rates as of 2026** (verify against current RO law — Law 296/2023 has
moved several rates):

- **Standard RO VAT:** 21% (changed in 2025 from 19%)
- **Reduced rates:** 11% (food, books, water) — confirm commodity eligibility
- **Other EU member states:** different rates — pull from profile country setting

Surface the rate used in the output. Operators sometimes catch a wrong rate
before the declarant does.

### HS code guidance

When the operator gives a commodity description without an HS code:

1. Suggest a likely 6-digit HS (international) and 8-digit TARIC (EU)
2. Always caveat: *"This is indicative. The licensed declarant is responsible
   for the final classification (UCC Art. 18). For binding tariff information,
   apply for a BTI."*
3. If the commodity is borderline between two HS positions, present both with
   the duty rates side by side so the operator sees the difference.

Common freight-relevant HS pairs:

| Description | Likely HS | Notes |
|---|---|---|
| Electric vehicles, complete | 8703.80 | EV-specific |
| Lithium-ion batteries, packed with equipment | 8507.60 | UN 3481 DG check |
| Lithium-ion batteries standalone | 8507.60 | UN 3480 — DG, restricted air |
| Auto spare parts, generic | 8708.99 | Subject to anti-dumping if China origin |
| Plastic toys | 9503.00 | CE marking + EN 71 + REACH SVHC |
| EVA footwear | 6402.99 | Standard duty, no extra cert usually |
| Lightning cables / chargers | 8504.40 | CE + RoHS + EU radio equipment directive |
| Apparel | 6101–6217 | Origin + GSP eligibility check |

## Export workflow

### Required documents (baseline)

- [ ] Commercial Invoice
- [ ] Packing List
- [ ] Transport document (AWB / B/L / CMR)
- [ ] Export declaration (filed by declarant via NCTS-export or AES)
- [ ] EORI number of the exporter
- [ ] Power of Attorney if broker-filed

### Conditional (by destination / commodity)

| Trigger | Document |
|---|---|
| Preferential origin claim by buyer | EUR.1 / EUR-MED / Origin Declaration |
| Dual-use items | Export licence + end-use statement |
| Military / defence | Specific export authorisation |
| Cultural property | UNESCO / member state authorisation |
| Endangered species | CITES |
| Sanctioned destinations | Compliance screening — operator must verify |

### EXS / ENS

For exports to certain destinations or transit, the Exit Summary Declaration
or Entry Summary Declaration may be required. Flag based on destination.

### ICS2 (Import Control System 2)

For all goods entering the EU by air, sea, road, rail — release 3 covering
all modes is live as of 2025. The pre-loading and pre-arrival declarations
are the carrier's responsibility, but the data originates with shipper /
forwarder. Surface this when drafting outbound shipments to the EU.

## Transit (T1 / T2 / NCTS)

For goods moving across EU territory under customs control without being
released for free circulation:

- [ ] T1 — non-Union goods
- [ ] T2 — Union goods moving via non-Union territory
- [ ] Filed via **NCTS-RO** (RO entry) or equivalent member-state NCTS
- [ ] **Guarantee** required — comprehensive guarantee waiver if AEO
- [ ] Office of departure / Office of destination on the declaration

DCS-style operations sometimes need independent T1/T2 capability when the
shipper isn't AEO and the broker is slow. See `legal-data-hunter` connector
(roadmap) for AEO partner forwarding.

## Output format

```
CUSTOMS CHECKLIST — [IMPORT / EXPORT / TRANSIT]
Direction:        [direction]
Origin:           [country]
Destination:      [country]
Mode:             [air / sea / road / rail]
Commodity:        [description]
HS code:          [code] (indicative — declarant confirms)
Declared value:   [amount] [CCY]
Incoterms:        [term + place]
─────────────────────────────────────────

REQUIRED (baseline)
  □ Commercial Invoice
  □ Packing List
  □ Transport document ([AWB / B/L / CMR])
  □ [DV1 if value > €20,000]
  □ EORI of [importer/exporter]
  □ Power of Attorney to broker

CONDITIONAL (commodity / origin)
  □ [doc 1]
  □ [doc 2]
  ...

ROMANIA-SPECIFIC
  □ [e-Transport — if RO border]
  □ [other RO items]

DUTY + VAT (indicative)
  Customs Value:    [amount] [CCY]   (= invoice + freight + insurance to EU)
  TARIC duty rate:  [%]
  Customs Duty:     [amount]
  Excise:           [amount or N/A]
  VAT Base:         [amount]
  VAT @ [%]:        [amount]
  ─────────────────
  Total at clearance: [amount]
─────────────────────────────────────────

DISCLAIMER
This checklist is indicative only. Classification, valuation, and filing
are the responsibility of the licensed customs declarant under UCC Art. 18.
For binding classification, apply for a Binding Tariff Information (BTI).
Duty rates and VAT rates verified against [date]; recheck before filing.

Generated by Claude for Freight (freight-forwarding plugin v1.0.0)
```

## Bilingual

If the operator types in Romanian, respond in Romanian. Key terms:

| EN | RO |
|---|---|
| Customs declaration | Declarație vamală |
| Customs value | Valoare în vamă |
| Customs duty | Taxă vamală |
| VAT base | Bază TVA |
| Import VAT | TVA import |
| HS code / TARIC | Cod vamal / TARIC |
| Declarant | Declarant vamal |
| Customs broker | Comisionar în vamă |
| Power of Attorney | Împuternicire / procură |
| AEO | OEA (Operator Economic Autorizat) |
| Preferential origin | Origine preferențială |
| Transit | Tranzit |
| Free circulation | Punere în liberă circulație |

## What this skill does not do

- It does not file. Filing is the declarant's act.
- It does not give binding tariff classifications. BTI is the formal route.
- It does not screen for sanctions or dual-use ECCN classification — that's
  a compliance skill; see the `pricing-procurement` plugin's compliance
  module (roadmap).
- It does not handle DG-specific paperwork (Shipper's Declaration for
  Dangerous Goods, ADR Transport Document). See `dangerous-goods` plugin.
