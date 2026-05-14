---
name: rfq-draft
description: Direct invocation of the rfq-drafting skill. Use when you want to draft an inbound or outbound RFQ email and don't want to wait for the skill to be inferred.
---

# /freight-forwarding:rfq-draft

Triggers the [`rfq-drafting`](../skills/rfq-drafting/SKILL.md) skill directly.

## Usage

```
/freight-forwarding:rfq-draft
```

Tell the skill which direction (inbound — replying to a customer; outbound —
asking a carrier) and provide:

- Origin, destination
- Cargo (or paste an existing thread)
- Special requirements (time-critical, ADR, DG, AOG, automotive line-stop)
- Recipient (carrier name or "use my Frankfurt agent")

## Examples

> Outbound RFQ to my Hong Kong agent, 3 pallets 120×80×120 cm, 800 kg each,
> mobile phones (HS 8517), EXW Shenzhen, ready Friday, air freight, please.

> Inbound — customer asks for a quote OTP→AMS, 1 pallet 100×80×120 cm, 250 kg,
> auto parts, DAP, ready today, time-critical for line stop.

## Notes

- Requires a populated practice profile (run cold-start first).
- For AOG / charter / OBC / DG-specific drafts, install the specialized
  plugin — `aog-response`, `air-charter`, `obc-nfo`, `dangerous-goods`. This
  plugin will draft a holding reply and flag.
