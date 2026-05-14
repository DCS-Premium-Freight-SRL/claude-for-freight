---
name: transport-doc
description: Direct invocation of the transport-documents skill. Use when you want to draft a CMR, AWB, HBL, MBL, Booking Confirmation, Proforma Invoice, or Packing List.
---

# /freight-forwarding:transport-doc

Triggers the [`transport-documents`](../skills/transport-documents/SKILL.md)
skill directly.

## Usage

```
/freight-forwarding:transport-doc
```

The skill will ask which document type if you don't specify. You can also
say it upfront:

> Draft a CMR for the shipment to Frankfurt tomorrow — 3 EUR pallets, 850 kg.

> AWB for OTP-DXB, our HAWB, 4 pallets 120×80×140, 1200 kg total, ready today.

> Generate booking confirmation for [REF].

## Document types supported

- CMR (road consignment note)
- AWB / HAWB / MAWB (air)
- HBL / MBL (ocean)
- CIM (rail — basic)
- Booking Confirmation
- Proforma Invoice
- Packing List

## Output options

- Markdown in chat (default)
- `.docx` if the `docx` skill is installed
- `.pdf` if the `pdf` skill is installed
- Save to Google Drive / OneDrive if connected — the skill will ask

## Notes

- The skill drafts; it does not issue. Originals require operator sign and
  stamp, and in some cases carrier counter-signature.
- DG-specific documents (DGD, Shipper's Declaration for DG) are not in this
  plugin — see `dangerous-goods` plugin.
