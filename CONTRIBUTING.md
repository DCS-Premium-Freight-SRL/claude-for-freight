# Contributing

The bar for merging is *"another forwarder could fork this and run it tomorrow
without calling us."* If you can't honestly say that about your contribution,
it needs more work before it's ready.

## Before you open a PR

**Open an issue first.** Describe the workflow you want to automate, the role
of the person who would run it (commercial / ops / customs / DG / pricing), and
one realistic prompt the way an operator actually types — not the way a product
manager writes a user story.

Realistic:

> *"fă-mi un AWB pt shipmentul de mâine la Frankfurt, 3 pal euro, 450kg total, expeditor e DCS"*

Not realistic:

> *"As a freight forwarder, I would like to generate an air waybill so that I can document the shipment."*

The plugins triage on real-world phrasing. If you can't write the prompt the way
an operator types it, the skill description won't trigger.

## The three patterns

Pick the one that fits the workflow before you start writing.

**Pattern A — Prompt only.** The skill is a Markdown file with a clear procedure
and templates. Used for: RFQ drafting, capability decks, email replies, escalation
memos. No code, no MCP. Fastest to ship, easiest for non-developer ops staff to edit.

**Pattern B — Prompt plus scripts.** The skill is Markdown with `scripts/` —
deterministic calculations, file validators, format converters. Used for: cargo
calculations, customs duty estimates, rate comparison matrices. Anything where
the answer must be reproducible and auditable, not generative.

**Pattern C — Skill plus MCP.** The skill drives an external system through an
MCP connector. Used for: anything that reads from Gmail, writes to Drive, queries
a TMS, pulls a rate from cargo.one. Goes in the plugin only if the MCP server is
in `connectors/`; vendor-specific server code stays out of the plugin.

Don't mix B and C unless the workflow genuinely needs both. A cargo calculation
is Pattern B (it has formulas). Pulling the resulting AWB from Drive is Pattern C
(it talks to an external system). Two skills, one plugin.

## Skill description rules

The description is the only thing Claude sees when deciding whether to load
the skill. The body of `SKILL.md` doesn't matter if the description doesn't
trigger.

- **Maximum 1024 characters.** Verify before opening the PR.
- **Pushy.** Vague descriptions lose to Claude's general judgment every time.
  Be specific about what the skill covers and use trigger keywords.
- **Bilingual where the operation is bilingual.** Romanian and English trigger
  keywords for plugins built around RO/EU operations. Add other languages where
  the practice profile actually uses them.

Verify the length:

```bash
python3 -c "
import re
c = open('SKILL.md').read()
m = re.search(r'description: >(.*?)(?=\nmetadata:|\n\w+:)', c, re.DOTALL)
d = re.sub(r'\n\s+', ' ', m.group(1).strip())
print(f'{len(d)} chars — {\"ok\" if len(d) <= 1024 else \"OVER LIMIT\"}')"
```

## What belongs in this repo

- **General patterns** that work across forwarders (RFQ drafting, cargo calc,
  CMR/AWB generation, customs checklist).
- **Industry-standard workflows** (IATA DG, ADR, AOG response, OBC dispatch,
  ICS2 entry summary declaration).
- **Open MCP connectors** to public standards (IATA Cargo XML, ONE Record,
  CBAM transitional registry) and to systems where the API is publicly documented.

## What does not belong

- **Your buy rates, your customer list, your agent network.** That's your moat,
  not ours. The practice profile is for that and stays local to your install.
- **Vendor-specific plugin code.** A CargoWise-only plugin is not useful to
  the 70% of forwarders who don't use CargoWise. CargoWise integration belongs
  as an MCP connector in `connectors/`, opt-in.
- **Anything you don't have legal clearance to release.** If your customs
  playbook is under NDA with a Tier-1 OEM, don't put it here. We'll know,
  and we'll have to revert it.

## Review process

PRs are reviewed by DCS Premium Freight ops and engineering. We're not a
foundation. There is one maintainer responding to issues as time allows.
Expect 3–10 business days for first response on a substantive PR. Trivial
fixes (typos, broken links) are usually faster.

If you've built something substantial and want to be added as a co-maintainer
of a plugin you contributed, open an issue with the proposal. We'd rather have
domain experts owning their plugins than a single bottleneck.

## Code of conduct

Be useful. Be specific. Don't waste each other's time. If you wouldn't say it
to a colleague across the desk, don't put it in a PR comment.

---

By contributing you agree your contribution is licensed under Apache 2.0 —
the same license as the rest of the repo. See [`LICENSE`](LICENSE).
