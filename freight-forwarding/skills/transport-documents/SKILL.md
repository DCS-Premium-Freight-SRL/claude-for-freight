---
name: transport-documents
description: >
  Draft transport documents — CMR (road), AWB / HAWB / MAWB (air), HBL / MBL
  (sea), CIM (rail), Booking Confirmation, Proforma Invoice, Packing List.
  Reads shipment details from operator input, calls the cargo-calculation
  skill when figures are missing, fills standard fields, surfaces what's
  missing. Generates .docx or .pdf if the docx/pdf skill is available.
  TRIGGER: CMR, AWB, HAWB, MAWB, HBL, MBL, conosament, Air Waybill, Bill of
  Lading, booking confirmation, proforma invoice, packing list, fă un CMR,
  generate AWB, document transport, transport document, scrisoare de trăsură,
  CIM rail consignment, FCR, FBL.
  Also: "fă-mi un CMR pentru", "draft an AWB for the shipment to", "generate
  a booking confirmation", "scrie un proforma", "îmi trebuie HBL la marfa",
  "pregătește documentele de transport".
metadata:
  author: DCS Premium Freight SRL
  version: 1.0.0
  pattern: A+C  # Prompt + optional MCP save to Drive
---

# Transport documents

Generates the standard transport documents for a shipment. One skill, multiple
document types — the operator picks which one (or asks for "the documents",
in which case ask which mode and infer the set).

## Read the profile first

Pull from `CLAUDE.md`:

- Company identity (sender on CMR / shipper on AWB / forwarder on HBL)
- Reference format
- Default disclaimer text on Proforma / Packing list
- HBL issuance policy (own house BL vs carrier MBL only)
- CMR issuance policy (own vs use carrier's)
- Sign-off block

## Inputs to gather

For every document:

- Shipment reference (use profile format if not given)
- Shipper / Consignee (full address)
- Origin / Destination (city, country, post code, IATA/port)
- Cargo (pieces, dimensions, GW, CBM, CW — call `cargo-calculation` if missing)
- Commodity description
- Incoterms (term + named place)
- Currency and declared value
- Special handling instructions

Then document-specific fields below.

## CMR (Road Consignment Note)

### Required boxes (CMR convention, 24 boxes)

1. Sender (expeditor) — name, address, country
2. Consignee (destinatar) — name, address, country
3. Place of delivery — full address
4. Place and date of taking over the goods
5. Documents attached — invoice no., packing list, EUR.1, etc.
6. Marks and numbers
7. Number of packages
8. Method of packing
9. Nature of the goods (description)
10. Statistical number / HS code
11. Gross weight (kg)
12. Volume (m³)
13. Sender's instructions
14. Stipulated cost (if applicable)
15. Carrier's reservations
16. Carrier name + plates + driver
17. Successive carriers (if any)
18. Carrier's reservations and observations
19. Special agreements
20. To be paid by — sender / consignee
21. Country / place / date of CMR establishment
22. Sender signature + stamp
23. Carrier signature + stamp
24. Consignee signature + stamp (signed at delivery)

### Output format

```
CMR — INTERNATIONAL CONSIGNMENT NOTE
Reference: [REF]
Established at: [place], [date]

┌── 1. Sender ─────────────────────────────────┐
│ [Name]                                       │
│ [Address]                                    │
│ [Country / VAT]                              │
└──────────────────────────────────────────────┘

┌── 2. Consignee ──────────────────────────────┐
│ [Name]                                       │
│ [Address]                                    │
│ [Country / VAT]                              │
└──────────────────────────────────────────────┘

3. Place of delivery:  [address]
4. Place / date taking over:  [origin], [date]
5. Documents attached:  [list]

┌── 6-12. Cargo ───────────────────────────────┐
│ Marks/Nos:      [pallet labels / SKUs]       │
│ Packages:       [N] × [type]                 │
│ Description:    [commodity]                  │
│ HS code:        [code]                       │
│ Gross weight:   [X] kg                       │
│ Volume:         [X.XX] m³                    │
└──────────────────────────────────────────────┘

13. Sender's instructions:  [any]
15. Carrier reservations:   [field for carrier]
19. Special agreements:     [any — Convention CMR applies]
20. Payment of carriage:    [PREPAID / COLLECT]

21. [Country / place / date]
22. Sender signature
23. Carrier signature (name + plates + driver)
24. Consignee signature (at delivery)
```

### CMR rules to enforce

- Number of original CMRs: **three** (sender, consignee, carrier).
- The CMR is *prima facie* evidence of the contract of carriage under the
  CMR Convention 1956 — flag this in the disclaimer line if drafting for
  a new operator.
- Successive carriers (box 17) — if hand-over to another carrier mid-route,
  must be on the document. Ask the operator if relevant.
- For RO domestic: align with Romanian Law 38/2008 transposition. CMR is
  bilingual RO/EN by convention.

## AWB / HAWB / MAWB (Air Waybill)

### Required fields

- Shipper name + address + account number (IATA or carrier-specific)
- Consignee name + address (cannot be "to order" — AWB is non-negotiable)
- Issuing carrier + AWB number prefix (3-digit carrier code + 8-digit serial)
- Airport of departure (IATA code)
- Airport of destination (IATA code)
- Routing — via airports + carrier codes
- Accounting information
- Currency
- Charge code (PP — prepaid / CC — collect)
- Other charges (PP/CC)
- Declared value for carriage
- Declared value for customs
- Amount of insurance
- Handling information (e.g., "KEEP UPRIGHT", "PERISHABLE", "TIME-SENSITIVE",
  "EXPEDITE")
- Pieces / RCP (rate class) / chargeable weight / commodity item number
- Gross weight (kg) and method (K or L)
- Rate / charge
- Total

### House vs Master

If the operator is issuing their own HAWB (house air waybill):

- Use the operator's IATA agent code prefix
- Shipper = the actual exporter
- Consignee = the actual importer
- Reference back to the MAWB the carrier will issue

If issuing master only — carrier's AWB — operator is just preparing the data
for the carrier to fill. Output a pre-fill sheet rather than a styled AWB.

### Time-critical / AOG markers

Auto-add to Handling Information:

- `EXPEDITE` for any time-critical
- `AOG SHIPMENT — IMMEDIATE ATTENTION` for AOG
- `DG — see DGD` if any dangerous goods present (the actual DGR work is
  the `dangerous-goods` plugin's job)

## HBL / MBL (Ocean Bill of Lading)

### Required fields

- Shipper / Consignee / Notify party (full addresses)
- Ocean vessel + voyage number
- Port of loading / Port of discharge
- Place of receipt / Place of delivery
- Container numbers + seal numbers
- Marks and numbers
- Description of goods
- Gross weight (kg) and measurement (m³)
- Number of originals issued
- Freight terms (prepaid / collect)
- Place and date of issue
- Signed by

### Type of B/L

Ask the operator if not specified:

- **Original (negotiable)** — three originals, surrendered for cargo release
- **Sea Waybill / Express Release** — non-negotiable, no surrender required
- **Telex Release** — original surrendered at origin, electronic release
  at destination

This is a high-stakes decision — surface it explicitly:

> "Sea waybill vs original — sea waybill faster but loses cargo control;
> original required if shipper wants to retain title until payment. Which?"

### House vs Master (same logic as AWB)

If the operator issues an HBL, the carrier issues an MBL; the HBL references
the MBL. If operator only handles MBL, output a pre-fill sheet.

## Booking Confirmation

The internal-facing acknowledgement to the customer that the shipment is
booked. Lighter than a B/L — informational, not a contract of carriage.

```
BOOKING CONFIRMATION
Reference: [REF]
Date: [DD.MM.YYYY]

Shipper:         [details]
Consignee:       [details]
Notify:          [if different]

Origin:          [city / airport / port]
Destination:     [city / airport / port]
Mode:            [AIR / SEA FCL / SEA LCL / ROAD / RAIL / MULTIMODAL]
Carrier:         [name] / [flight, vessel, truck ref] / ETD [date] / ETA [date]

Cargo:           [pieces × dims] / GW [X] kg / CBM [X] m³ / CW [X] kg
Commodity:       [description]
HS code:         [code if known]

Incoterms:       [term + place]
Freight terms:   [prepaid / collect / as agreed]
Estimated rate:  [optional — include if quote was binding]
Validity:        [if rate included]

Special:         [time-critical / temperature / DG / fragile / etc.]

Pre-alerts to be sent on departure and arrival.

Issued by:
[Sign-off]
[Company + reference]
```

## Proforma Invoice

For customs / advance payment — not a final commercial invoice.

```
PROFORMA INVOICE
Reference: [REF]
Date: [date]

Seller:          [legal name + VAT + address]
Buyer:           [legal name + VAT + address]

Incoterms:       [term + named place]
Country of origin: [country]
Country of destination: [country]
Mode of transport: [air / sea / road / rail]
Currency:        [CCY]

┌─ Description ──────────┬─ HS ────┬─ Qty ─┬─ Unit ─┬─ Total ──┐
│ [item]                 │ [code]  │ [n]   │ [u]    │ [amount] │
└────────────────────────┴─────────┴───────┴────────┴──────────┘

Subtotal:        [amount]
Freight:         [amount, if included]
Insurance:       [amount, if included]
Total:           [amount]

PURPOSE: [FOR CUSTOMS PURPOSES ONLY — NOT FOR PAYMENT  /  FOR ADVANCE PAYMENT]

[Standard disclaimer from profile]

[Sign-off]
```

## Packing List

Pure dimensional document — no values. Required for customs clearance and
warehouse handover.

```
PACKING LIST
Reference: [REF]
Date: [date]

Shipper:         [name + address]
Consignee:       [name + address]
Origin:          [city]
Destination:     [city]

Shipping marks:  [labels visible on packaging]

┌─ Pkg ─┬─ Type ────┬─ Contents ─────────┬─ Qty ─┬─ NW kg ─┬─ GW kg ─┬─ L×W×H cm ─┐
│ 1     │ Pallet    │ [SKU / desc]       │ [n]   │ [x]     │ [y]     │ [LxWxH]    │
│ 2     │ Carton    │ [SKU / desc]       │ [n]   │ [x]     │ [y]     │ [LxWxH]    │
└───────┴───────────┴────────────────────┴───────┴─────────┴─────────┴────────────┘

Totals:          [N] packages / [NW kg] net / [GW kg] gross / [CBM m³]

[Sign-off]
```

## Output format and handoff

- Generate the document as Markdown in the operator's chat first
- Offer to convert to `.docx` (call the `docx` skill) or `.pdf` (call the
  `pdf` skill) if installed
- If Google Drive MCP is connected: offer to save in the shipment folder
  using the structure:

```
[Company] / Shipments / [REF] /
  ├── Commercial/
  │   ├── [REF]-Proforma-Invoice.docx
  │   └── [REF]-Packing-List.docx
  ├── Transport/
  │   ├── [REF]-CMR.docx
  │   ├── [REF]-HBL.docx
  │   └── [REF]-Booking-Confirmation.docx
  └── Customs/
      └── [REF]-Customs-Documents/
```

- Always ask before saving: *"Save to Drive in the [REF] folder?"*

## What this skill does not do

- It does not file the document. CMR isn't enforceable until signed; AWB isn't
  active until the carrier issues. The operator does the issuance.
- It does not issue *original* documents on the operator's behalf. Outputs
  are drafts on the operator's stationery.
- It does not draft DG-specific documents (DGD, Shipper's Declaration for
  Dangerous Goods). The `dangerous-goods` plugin handles DG — installed
  separately.
- It does not handle CIM rail consignment notes in depth — basic CIM
  generation is here, but rail-heavy operations should use a dedicated
  rail plugin (roadmap).
