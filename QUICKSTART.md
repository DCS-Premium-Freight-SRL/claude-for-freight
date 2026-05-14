# Quickstart

Five minutes to your first cargo calculation in Claude. Fifteen if you do the cold-start
interview properly (recommended — every other skill is sharper after it).

---

## What you need

- **Claude Code** (terminal) or **Claude Cowork** (desktop). Either works.
- **An Anthropic API key.** Pricing at [anthropic.com/pricing](https://anthropic.com/pricing).
- **One of your real RFQs** from the last two weeks. Doesn't matter which. The cold-start
  interview is much better with one real example than five hypothetical ones.

You do **not** need a TMS, a customs broker login, or a cargo.one account to start.
Those plug in later through `connectors/`.

---

## 1. Clone the repo

```bash
git clone https://github.com/dcs-freight/claude-for-freight.git
cd claude-for-freight
```

## 2. Add it as a Claude Code marketplace

In Claude Code, type `/plugin marketplace add ` (with a trailing space), then drag the
unzipped `claude-for-freight` folder onto the terminal — it'll fill the path in.
Press Enter.

Or type the path directly:

```
/plugin marketplace add /Users/you/code/claude-for-freight
```

## 3. Install the plugin

Start with `freight-forwarding`. It's the most general — covers RFQ, cargo calc,
transport documents, customs checklist. The specialized plugins (`aog-response`,
`air-charter`, `dangerous-goods`) layer on top once you've got the general one working.

```
/plugin install freight-forwarding@claude-for-freight
```

You'll be asked whether to install for **this project only** (project scope) or
**all projects** (user scope). For most desk operators: user scope. Project scope
blocks the plugin from reading files outside the project folder — your rate sheets
in Downloads, the customer's PO sitting on Desktop, the carrier's PDF in your email
attachments folder. None of that works in project scope.

## 4. Restart Claude Code

Close it. Reopen it. **This step is not optional** — the plugin isn't live until
you restart.

## 5. Run the cold-start interview

```
/freight-forwarding:cold-start-interview
```

Takes ten to fifteen minutes. It will ask:

- Your company name, EORI, default sender details
- The lanes you cover and the modes (air / sea / road / rail / multimodal)
- Your standard incoterm assumptions when the customer doesn't specify
- The format and prefix of your shipment reference numbers
- Your escalation thresholds (who has to sign off on what)
- Which agents you trust on which lane
- Which TMS, customs platform, and rate platforms you actually use

You can also do a **quick start** (two minutes) — just the company identity and one
example RFQ. Everything else fills in over the next few uses as you correct it.

The output is `CLAUDE.md` in your plugin config directory:

```
~/.claude/plugins/config/claude-for-freight/freight-forwarding/CLAUDE.md
```

You can open it. You can edit it. You can re-run setup. You can tell a skill to
update it — "remember that we now use DHL Global Forwarding as our primary on the
Frankfurt lane." Every skill in the plugin reads from this file before it does
anything else.

---

## 6. Run your first command

Try one of:

```
/freight-forwarding:cargo-calc
/freight-forwarding:rfq-draft
/freight-forwarding:transport-doc
/freight-forwarding:customs-checklist
```

Or just talk to it:

> *"alo cât iese chargeable pe 4 colete 60x40x30 fiecare 18kg pe air freight"*

The skill triggers on the natural-language phrasing too. You don't have to remember
the slash command.

---

## 7. Connect the systems you actually use (optional)

The plugin works without connectors — you'll just paste rates and dimensions by hand.
To stop pasting, hook up the connectors:

- **Gmail / Outlook** — so the plugin can read the customer's RFQ thread and draft
  the reply directly
- **Google Drive / OneDrive** — so generated CMR / AWB / Booking confirmations save
  to the right shipment folder
- **cargo.one / WebCargo** — so air rates are pulled live
- **Your TMS** — see [`connectors/`](connectors/) for the available specs

In Cowork: Settings → Connectors → add the relevant ones.
In Claude Code: the plugin's `.mcp.json` lists the MCP servers; you'll be prompted
to authorize on first use.

---

## Troubleshooting

**"Command not found" after install.** You forgot step 4. Restart Claude Code.

**"Run setup first" when you try a command.** Run
`/freight-forwarding:cold-start-interview` first. The skills refuse to run without
a practice profile because their output would be generic — which is worse than no
output, because somebody might trust it.

**"I can't read [file]".** Almost always means you installed project-scoped and
the file is outside the project folder. Uninstall and reinstall user-scoped
(see step 3).

**Cargo calculations look wrong for my region.** The defaults use IATA volumetric
divisor 6000 for air and the standard EU 2.4 m trailer width for LDM. If your
operation uses 5000 (some couriers do) or different trailer dimensions, tell the
cold-start interview, or edit the practice profile after the fact.

**The rates the plugin suggests are nowhere near reality.** The plugin does not
know your buy rates. It does cargo calculations and document drafting. Rate
suggestions only appear if you connect a rate platform (cargo.one, WebCargo) or
load a rate sheet into the cold-start interview.

---

## What to do next

Read [`docs/architecture.md`](docs/architecture.md) — the three-layer pattern and
why it works. Look at [`docs/authority-matrix.md`](docs/authority-matrix.md) — the
approval gates that make autonomous operation safe at the desk. Then, when you're
ready for the patterns that run while you're asleep, look at the managed agents
in `freight-forwarding/agents/`.
