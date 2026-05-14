---
name: rfq-drafting
description: >
  Draft inbound and outbound RFQ emails for freight quotations. Inbound = a
  customer asks the forwarder for a quote; outbound = the forwarder asks a
  carrier or agent for a rate. Uses the practice profile for tone, sign-off,
  reference format, and lane-specific agent picks. Reads attached threads
  and dimensions when Gmail or Outlook MCP is connected.
  TRIGGER: RFQ, quote request, cerere de ofertă, ofertă freight, draft an
  RFQ, scrie un RFQ, write an RFQ email, email partener, email transportator,
  trimite la agent, send to carrier, rate request, freight quote email,
  cotație, request a rate, ask the agent.
  Also: "scrie un email la agentul din", "draft RFQ to", "cerere ofertă pe lane-ul",
  "trimite cerere rată la", "email către carrier pentru shipmentul de", "follow up
  on the quote", "chase the rate".
metadata:
  author: DCS Premium Freight SRL
  version: 1.0.0
  pattern: A  # Prompt only
---

# RFQ drafting

Drafts the email that asks for or answers a freight quote. Direction matters:
inbound and outbound have different structures, different tones, and different
required fields.

## Read the profile first

Pull from `CLAUDE.md`:

- Company identity (legal name, address, contact, sign-off block)
- Reference format
- Default incoterms (inbound and outbound)
- Default currency
- Agent network for lane suggestions
- Lane preferences and time-critical defaults
- Example RFQ captured at cold-start (for tone)

If no profile, prompt the operator to run
`/freight-forwarding:cold-start-interview` first. Skill refuses to draft
without identity, because a draft with placeholder sender details is worse
than no draft.

## Classify direction

First decide: is this **inbound** (we're answering a customer) or **outbound**
(we're asking a carrier/agent)?

Signals:
- "draft RFQ to [an agent name we recognize]" → outbound
- "respond to RFQ from [customer]" or pasted customer enquiry → inbound
- "ask Aramex / DHL / our Frankfurt agent for a rate" → outbound
- "the customer wants a quote on" → inbound

If genuinely ambiguous, ask once: *"This for the carrier (asking for a rate)
or for the customer (replying with a quote)?"*

## Outbound RFQ — workflow

### Required fields

Gather, then draft:

- **Origin** — city + country + (airport IATA / port / postal code)
- **Destination** — same
- **Cargo** — pieces × dimensions, gross weight (compute chargeable via the
  `cargo-calculation` skill if not already done)
- **Commodity** — generic description; HS code if known
- **Incoterms** — from profile default if customer didn't specify
- **Ready date** or "ready immediately"
- **Special requirements** — time-critical, ADR, temperature, automotive
  line-stop, OBC eligible, DG class
- **Carrier(s)** to send to — from profile agent network, or operator-named

### Lane-specific routing

When the lane matches a lane in the profile, suggest the trusted agents for
that lane in priority order. *"For OTP–FRA you usually go to [Agent A] then
[Agent B] — send to both for benchmark?"*

If the lane isn't in the profile, ask: *"New lane — who should I send to?
Want to add this to the profile?"*

### Time-critical flagging

If the operator marks the shipment as time-critical, AOG, or automotive
line-stop, or the deadline is under 24h:

- Add `[URGENT]` or `[AOG]` in the subject line
- State the deadline explicitly (date *and* time)
- Ask for ETD/ETA confirmation, not just rates
- For AOG specifically: hand off to the `aog-response` plugin if installed —
  this plugin is general; AOG has a specialized workflow

### Template (EN)

```
Subject: RFQ – [Origin IATA] > [Destination IATA] | [Ready date] | Ref [REF]

Dear [Agent name],

Please quote your best rate for the below:

Origin:           [city, country, IATA/port]
Destination:      [city, country, IATA/port]
Cargo:            [pieces] × [L × W × H cm] / GW [X] kg / CBM [X] / CW [X] kg
Commodity:        [description] (HS [code if known])
Incoterms:        [term + named place]
Ready date:       [date, time if relevant]
Mode:             [air / sea / road]
Special:          [time-critical / ADR / temp / none]

Please include:
- All-in rate (currency: [CCY])
- Transit time door-to-door
- Earliest pickup slot
- Validity period of the rate
- Any surcharges (fuel, security, war risk if applicable)

Reply by [deadline if any].

Best regards,
[Sign-off from profile]
[Company]
[Reference: REF]
```

### Template (RO)

```
Subiect: Cerere ofertă – [Origine] > [Destinație] | [Data] | Ref [REF]

Bună ziua [Nume],

Vă rog cea mai bună cotație pentru:

Origine:           [oraș, țară, IATA/port]
Destinație:        [oraș, țară, IATA/port]
Marfa:             [colete] × [L × W × H cm] / GB [X] kg / CBM [X] / GT [X] kg
Comoditate:        [descriere] (HS [cod dacă e cunoscut])
Incoterms:         [termen + loc]
Data gata:         [data]
Mod:               [aerian / maritim / rutier]
Special:           [time-critical / ADR / temperatură / nimic]

Vă rog includeți:
- Tarif total (monedă: [CCY])
- Timp de tranzit door-to-door
- Cel mai apropiat slot pickup
- Validitate ofertă
- Suprataxe (fuel, security, war risk dacă e cazul)

Răspuns până la [deadline].

Cu stimă,
[Semnătura din profil]
[Companie]
[Ref: REF]
```

## Inbound RFQ — workflow

### Required fields (read from the customer's email)

Extract or ask for:

- Shipper / Consignee (full address)
- Cargo (pieces, dimensions, weight)
- Commodity
- Incoterms
- Ready date / required delivery date
- Special requirements
- Volume frequency (one-off or recurring; affects rate strategy)

### Quote response structure

If the operator wants to send the customer a quote *now* (rates already
known), draft as:

```
Subject: Quote – [Origin] > [Destination] | [Customer ref or our REF]

Dear [Customer name],

Thank you for your enquiry. We are pleased to quote:

Service:          [air / sea / road / multimodal]
Origin > Dest:    [details]
Cargo:            [pieces / weight / CBM / CW]
Transit time:     [X] business days door-to-door
All-in rate:      [amount] [CCY] — [terms: all-in / + duty + VAT / etc.]
Validity:         [date]
Pickup slot:      [date range]

The rate includes: [pickup / freight / arrival handling / delivery / customs as applicable]
The rate does not include: [duty, VAT, demurrage beyond X days, abnormal handling]

[Standard disclaimer from profile]

Please confirm by [date] to secure the slot.

Best regards,
[Sign-off]
```

If rates aren't known yet — the operator is acknowledging receipt only —
draft an interim:

```
Subject: Re: Your RFQ – received, quoting

Dear [Customer name],

Confirming receipt of your enquiry on [reference]. We are sending to our
carrier panel and will revert with a fully validated quote by [date/time].

In the meantime, please confirm:
- [Any missing fields]
- [Validity of the cargo info — same shipment as previously, or new specs]

Best regards,
[Sign-off]
```

### Negotiation reply

If the customer pushed back on a previous quote, draft a counter that:

- Acknowledges the price point without conceding
- Surfaces what's negotiable (transit time tradeoff, mode swap, consolidation)
- Holds firm on what isn't (margin floor from profile, if defined)

Refer to the captured example RFQ in the profile for tone. Do **not** invent
a softer or harder tone than the operator's actual style.

## Cargo calculation handoff

If the operator gives dimensions but no chargeable weight, run the
`cargo-calculation` skill first, then embed the result in the email.
Don't draft an RFQ with missing cargo metrics.

## Gmail / Outlook integration

If Gmail or Outlook MCP is connected:

- Offer to draft directly in the operator's inbox (as a draft, not sent)
- Search the recent thread for context — has this lane been quoted before
  by this agent? Mention it in the draft
- Tag with the reference so the agent's reply lands in the right thread

If MCP is not connected, present the email as text the operator can paste.

## What this skill does not do

- It does not bind the operator to a rate. Drafted quotes are drafts.
- It does not negotiate without supervision. Counter-offers are surfaced to
  the operator for review.
- It does not handle charter, OBC, DG, or AOG enquiries — those are
  high-stakes and route to the specialized plugins
  (`air-charter`, `obc-nfo`, `dangerous-goods`, `aog-response`). If those
  plugins aren't installed, draft a minimum-viable holding reply and flag
  to the operator that this needs specialist handling.
