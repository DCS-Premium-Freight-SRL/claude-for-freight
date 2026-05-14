# Connectors

MCP servers that bridge the plugins to the systems freight forwarders
actually run on. Plugin code stays vendor-neutral; vendor-specific glue
lives here, opt-in per operator.

> **Status: roadmap.** None of the freight-industry connectors below are
> shipped today. The plugins work without them — output falls back to
> text the operator copies into the system manually. As each connector
> is built, it lands in its own subfolder with a README, an `mcp.json`
> reference config, and (where licensable) a reference implementation.

## Why connectors and not plugin-side API clients

Three reasons, in order of how much each one matters in practice.

**Authentication once.** The operator authorizes a connector once. Every
plugin that needs that data uses the same connection. Otherwise you get
"this plugin wants Gmail access" four separate times because four
separate plugins ship their own Gmail client.

**Plugin code stays vendor-neutral.** The `aog-response` skill asks for
"rates on OTP–FRA, next 12 hours, AOG priority." Whichever rate connector
is installed answers. The skill doesn't change when the operator switches
from cargo.one to WebCargo.

**Connector code stays separable from plugin code.** Vendor changes the
API; the connector updates; the plugin doesn't notice. Vendor goes out of
business; you swap the connector; the plugin still works.

## Categories

### Productivity stack

Connected via Anthropic's standard MCP servers; no freight-specific glue
needed. Already supported by Claude Code and Cowork out of the box.

| Connector | What plugins use it for |
|---|---|
| **Gmail** | Read inbound RFQ threads, draft replies, send |
| **Outlook / Microsoft 365** | Same as Gmail for M365 operators |
| **Google Drive** | Save generated CMR / AWB / Booking / Proforma into shipment folders |
| **OneDrive / SharePoint** | Same as Drive for M365 operators |
| **Google Calendar** | Block pickup windows, delivery appointments, ETD/ETA milestones |
| **Slack / Teams** | Notification destination for managed agents (Authority Matrix escalations) |

### Rate platforms

| Connector | Status | Plugins |
|---|---|---|
| **cargo.one** | spec only | freight-forwarding, aog-response, air-charter |
| **WebCargo (Freightos)** | spec only | freight-forwarding |
| **CargoAi** | spec only | freight-forwarding |
| **xeneta** (benchmarking) | spec only | pricing-procurement |

The pattern: the connector exposes one tool, `get_rates`, with a normalized
signature (origin, destination, mode, cargo specs, urgency). The vendor-specific
authentication, request mapping, and response parsing live in the connector.

### TMS (Transport Management Systems)

| Connector | Status | Notes |
|---|---|---|
| **CargoWise One** (Descartes) | spec only | Read-only first; write later — booking, milestone, document log |
| **Riege Scope** | spec only | Read-only first |
| **StarCargo** | DCS reference | Read-only TMS for DCS Premium Freight; reference for the integration pattern |
| **Magaya** | spec only | Read-only first |
| **Logitude** | spec only | Read-only first |

TMS connectors are typically project-scoped (not user-scoped), so the
plugin can only read shipments belonging to the project the operator is
working in. Write access is opt-in per skill — booking confirmation is a
common write target; invoice issuance never is.

### Industry standards

These are the ones that matter long-term — they bypass vendor lock-in by
talking to industry-standard formats instead of vendor APIs.

| Connector | Standard | Status |
|---|---|---|
| **IATA Cargo XML / e-AWB** | IATA Cargo-XML | spec only |
| **IATA ONE Record** | ONE Record (REST/RDF) | spec only — the long-term ambition |
| **CargoMART (DSV/DHL e-AWB hubs)** | proprietary | spec only |
| **EDIFACT IFTMIN / IFTMBF / IFTSTA** | UN/EDIFACT | spec only — for legacy carrier integrations |

ONE Record is the most strategically important on the roadmap. As the
IATA-led successor to Cargo-XML lands across carriers and ground handlers
through 2026–2028, a forwarder with a working ONE Record connector
de-risks every future carrier integration to a configuration change.

### Customs & regulatory

| Connector | Authority | Status |
|---|---|---|
| **ANAF (RO customs / e-Transport / e-Factura)** | RO ANAF | spec only |
| **EU DG-TAXUD TARIC** | EU Commission | spec only (read-only TARIC lookup) |
| **EU CBAM Transitional Registry** | EU Commission | spec only |
| **EU ICS2 (Import Control System)** | EU Commission | spec only |
| **NCTS national gateways** | per member state | spec only |

These tend to be member-state-specific and slow to expose stable APIs. The
pattern: cache TARIC and CN lookups locally with periodic refresh, route
filing through the operator's existing broker software, but pull declaration
status back through the connector for tracking.

### Charter and OBC market data

| Connector | Source | Status |
|---|---|---|
| **Chapman Freeborn** | private | spec only — broker comms via email is the fallback |
| **Air Charter Service** | private | spec only |
| **Hunt & Palmer** | private | spec only |
| **Time:Matters / OBC market feeds** | private | spec only |

Charter is heavily relationship-driven; connector value is mostly in
post-flight cost reconciliation (chase backup invoices, match against
quoted) rather than in market sourcing.

### Visibility & tracking

| Connector | Source | Status |
|---|---|---|
| **project44** | private | spec only |
| **FourKites** | private | spec only |
| **MarineTraffic / VesselFinder** | private | spec only — for ocean vessel tracking |
| **FlightAware / FlightRadar24** | private | spec only — for aircraft tracking on charter |

## How to build a new connector

The bar is the same as for skills and plugins: another forwarder could
install your connector tomorrow and have it work against their account.

1. Open an issue describing the connector — which system, which tools the
   connector should expose, what authentication model.
2. Build the MCP server. Anthropic publishes the spec at
   [modelcontextprotocol.io](https://modelcontextprotocol.io). The HTTP
   transport is the path of least resistance for hosted vendors.
3. Submit a PR with:
   - `connectors/<name>/README.md` — what it does, authentication setup,
     known limitations
   - `connectors/<name>/mcp.example.json` — reference configuration that
     operators copy into their plugin `.mcp.json`
   - `connectors/<name>/IMPLEMENTATION.md` — for self-hosted connectors,
     the build and deployment notes; for hosted, the signup URL
4. Document the tools the connector exposes. The skill side then has a
   stable surface to call.

## A note on what's missing from the freight industry vs legal

Anthropic's `claude-for-legal` ships with around twenty connectors out of
the gate, including specialized legaltech (Ironclad, DocuSign, iManage,
Everlaw, CourtListener). That depth exists because the legaltech ecosystem
has API-first vendors with overlapping competitive pressure.

Freight forwarding is structurally behind. The major TMS vendors have
limited public APIs, by design — their business model depends on data
gravity. Rate platforms have APIs but they're rate-platform-specific. The
neutral standard the industry has agreed to (IATA ONE Record) is in early
production rollout as of 2026.

The opportunity, and the reason this repo exists, is that the connector
layer is the unbuilt half of the industry's AI tooling. Whoever builds
the open ONE Record connector first becomes the reference implementation
the rest of the industry forks. Whoever builds the open CargoWise read-only
connector first removes the biggest single objection to AI assistance on
forwarder desks ("we live in CargoWise; if the AI can't see CargoWise, it's
useless"). These are concrete, scoped, ship-in-a-quarter projects.

See [`../CONTRIBUTING.md`](../CONTRIBUTING.md). If you build one, we'll
review it.
