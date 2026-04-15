---
title: "Knowledge Bundle — Sales Plugin"
version: v0.1.0
---

# Knowledge Bundle

This directory is the sales plugin's **bundled research foundation** — a curated set of framework extracts that the plugin's six agents cite from, so that no citation in the output artifacts (`research/state-of-outbound.md`, `research/empirical-findings.md`, `research/category-context.md`, `scripts/cold-call-v1.md`, `scripts/objections-library.md`, `cadences/<name>.md`, `crm-setup.md`, `recordings-analysis/<recording-id>.md`) is hallucinated from training data.

**Rule:** agents cite from `knowledge/`, not from training data. If a framework is not in this directory, it is not citable. If a quote is not in the "Direct quotes" section of the relevant extract, it must not appear in a plugin output. This is not a style preference; it is a defensibility contract.

## Directory layout (v0.1.0 — flat after Phase 4 reconciliation)

The build originally created several semantic subdirectories (`methodologies/`, `objection-handling/`, `cold-email/`, `empirical-research/`, `blue-collar/`, `crm-tooling/`) but the agent and skill files all reference `knowledge/frameworks/` and `knowledge/reports/`. Phase 4 reconciled this by flattening into the two-directory structure the consumers expect.

```
knowledge/
├── README.md                                          (this file)
├── frameworks/                                        (16 extracts — the canonical sales canon)
│   ├── blount-fanatical-prospecting.md               scaffolding
│   ├── rackham-spin-selling.md                       evidence
│   ├── dixon-adamson-challenger-sale.md              vocabulary-only
│   ├── ross-predictable-revenue.md                   scaffolding
│   ├── keenan-gap-selling.md                         scaffolding
│   ├── sobczak-smart-calling.md                      scaffolding
│   ├── weinberg-new-sales-simplified.md              scaffolding
│   ├── braun-poke-the-bear.md                        scaffolding
│   ├── voss-never-split-the-difference.md            evidence
│   ├── blount-objections.md                          scaffolding
│   ├── jones-exactly-what-to-say.md                  scaffolding
│   ├── cialdini-influence.md                         evidence
│   ├── selling-to-skeptical-trades.md                scaffolding
│   ├── homeadvisor-angi-reverse-playbook.md          evidence
│   ├── hubspot-free-bootstrap-patterns.md            scaffolding
│   └── dialer-comparison-2026.md                     scaffolding
└── reports/                                           (2 extracts — empirical data refreshed periodically)
    ├── gong-100k-call-analysis.md                    evidence
    └── outreach-salesloft-cadence-research.md        evidence
```

**Tier count:** 6 evidence, 11 scaffolding, 1 vocabulary-only (18 total framework extracts plus this README).

## The tier system

Every extract file carries a `tier:` field in its YAML frontmatter with one of three values. The tier is load-bearing: when an agent cites a framework, it carries the tier forward, so a human reader of a final `cold-call-v1.md` script can see at a glance whether a given line is grounded in empirical data, in a disciplinary scaffolding, or in a shared vocabulary without predictive power.

### `tier: evidence`

The framework is empirically grounded — controlled studies, replicated findings, large-n field analyses, or published legal/regulatory records. Citing an `evidence` source carries real weight: the claim it backs is defensible against a skeptical reviewer.

In this bundle:
- `frameworks/rackham-spin-selling.md` — Huthwaite's 12-year, 35,000-call observational field study.
- `frameworks/voss-never-split-the-difference.md` — FBI Crisis Negotiation doctrine with peer-reviewed behavioral-science underpinnings.
- `frameworks/cialdini-influence.md` — Cialdini's principles drawn from peer-reviewed social psychology with hundreds of independent replications per principle.
- `frameworks/homeadvisor-angi-reverse-playbook.md` — direct customer-voice data + FTC enforcement record (Jan 2023, $7.2M settlement).
- `reports/gong-100k-call-analysis.md` — Gong's recorded-call analysis on 100k+ cold calls (300M+ total sales calls), with vendor-source caveat.
- `reports/outreach-salesloft-cadence-research.md` — Outreach.io and SalesLoft platform analyses on hundreds of millions of touch attempts, with vendor-source caveat.

### `tier: scaffolding`

The framework does not predict outcomes on its own, but it forces the founder to do work they would otherwise skip. The artifact it produces is valuable even if the underlying theory is weak. Most of the classic sales canon lives here — the books are not research, but they are disciplinary packaging that prevents lazy improvisation.

In this bundle:
- `frameworks/blount-fanatical-prospecting.md` — 30-Day Rule, Golden Hours, 5-step phone framework.
- `frameworks/ross-predictable-revenue.md` — Cold Calling 2.0, role specialization, named-account discipline.
- `frameworks/keenan-gap-selling.md` — current-state / future-state diagnosis, PIC questions.
- `frameworks/sobczak-smart-calling.md` — pre-call research checklist, Possible Value Propositions.
- `frameworks/weinberg-new-sales-simplified.md` — the 15-word Sales Story / power statement.
- `frameworks/braun-poke-the-bear.md` — Permission Opener, problem-first framing, no-pitch CTA.
- `frameworks/blount-objections.md` — RBO classification, the Ledge technique.
- `frameworks/jones-exactly-what-to-say.md` — 23 magic words / micro-phrases for transitions and closes.
- `frameworks/selling-to-skeptical-trades.md` — trades-vertical patterns: trust-before-pitch, recording-as-proof, founding-member framing, no-credit-card-day-1.
- `frameworks/hubspot-free-bootstrap-patterns.md` — HubSpot Free pipeline shape, custom fields, lead-score formula, `--apply` script pattern.
- `frameworks/dialer-comparison-2026.md` — manual / click-to-call / power-dialer / parallel-dialer decision tree with April 2026 pricing.

### `tier: vocabulary-only`

The framework sounds rigorous, but it does not predict outcomes. Its value is that it gives a team a **shared language** for things that would otherwise be argued about in vague terms. It is dangerous if treated as authority on buyer behavior, because it is not that. Vocabulary-only citations MUST carry the tier tag forward so the reader of the final script can see that the recommendation is vocabulary, not prediction.

In this bundle:
- `frameworks/dixon-adamson-challenger-sale.md` — Teach-Tailor-Take-Control opener pattern is useful; the 5 seller-profile segmentation is weak survey-derived factor analysis with no independent replication, and the headline "Challengers win 54%" claim does not survive scrutiny. The extract is honest about the split.

## How agents should use this directory — the 5 rules

1. **Read before citing.** When an agent needs a framework citation, it reads the relevant extract file in `knowledge/frameworks/` or `knowledge/reports/` first. It does not reach into training-data memory for quotes, page numbers, or framework details. If you find yourself wanting to cite a framework that is not in this directory, stop and write a `.sales/research/<slug>.md` extract first via `/sales:research`, then cite that — never cite from training data.

2. **Quote only from the "Direct quotes" section.** Every extract file has a "Direct quotes" section with attributed, fair-use quotes (≤200 words each). Agents may paste these into output verbatim with attribution. Quotes outside that section are out of scope. Several extracts in this v0.1.0 bundle mark quotes as paraphrases rather than verbatim — `script-writer` and `call-coach` should treat paraphrases as paraphrases (no quotation marks; cite as "paraphrase of [source]").

3. **Carry the tier forward.** When an agent cites a framework in `cold-call-v1.md`, `objections-library.md`, `cadences/<name>.md`, `crm-setup.md`, or `recordings-analysis/<recording-id>.md`, the citation format must include the tier tag. Example:

   > "Open with 'How have you been?' — `[Gong 100k cold-call analysis, tier: evidence]` — replicated 6.6x lift over baseline. Then transition with 'The reason I'm calling is...' — `[Blount, Fanatical Prospecting Step 3, tier: scaffolding]`."

   That is an honest citation. The reader knows at a glance which line rests on data and which line rests on a forcing-function framework.

4. **Refresh evidence on schedule.** Three files in this bundle are explicitly time-decaying: `reports/gong-100k-call-analysis.md` (vendor periodically republishes with newer samples), `reports/outreach-salesloft-cadence-research.md` (vendor blogs are updated), and `frameworks/dialer-comparison-2026.md` (pricing moves quarterly). The `sales-researcher` agent should re-verify these on every `/sales:research` run and update `.sales/research/empirical-findings.md` with any deltas. Stale snapshots older than 6 months should not be used without revalidation.

5. **The canonical bundle is frozen per release.** This `knowledge/` directory is shipped as part of plugin v0.1.0 and is not modified during normal use. The `sales-researcher` may add new extracts to `.sales/research/<slug>.md` inside the target project during `/sales:research`, but the canonical bundle is updated only on plugin version bumps. If a framework is not in this directory and not in `.sales/research/`, it is not citable.

## How a reader of `cold-call-v1.md` should use the tier tags

When reading the final cold-call script, look at the tier tag on every line citation. A script that leans heavily on `tier: vocabulary-only` lines (Challenger's 5 profiles, Sandler-style scripts, generic positioning mad-libs) is a feel-good consensus document, not a research-grounded plan. A script that leans on `tier: scaffolding` has disciplined process but does not claim to predict close rate. A script with meaningful `tier: evidence` citations (Gong opener data, Voss objection-handling, Cialdini triggers, Outreach/SalesLoft cadence shape, FTC reverse-playbook items) is the most defensible. The mix is up to the founder; the plugin's job is to make the mix legible so the founder can see exactly where the recommendation comes from.
