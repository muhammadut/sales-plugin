# Sales Plugin

**Research-grounded B2B outbound sales for Claude Code.** From empty directory to a defensible cold-outbound playbook in a working session — ICP, prospect-list strategy, scripts citing named frameworks, multi-touch cadences, CRM setup, role-play practice mode, and a recording-driven coaching loop.

## What it does

`sales` is a planning-and-iteration plugin for B2B outbound. It runs at the top of the outbound workflow, interviews you, dispatches 6 specialist agents, and produces a `.sales/` directory in your project containing every artefact a two-person team needs to start cold-calling and emailing. Every script line, objection response, and cadence step cites a named framework. Recordings flow through Deepgram or Azure Speech into a Gong-style coaching loop — solo founders get something like a sales-floor instrumentation stack at near-zero cost.

The plugin does NOT replace the human. It teaches the human to run the motion. Cold-calling is a craft; this plugin compresses 12 books of craft into a usable tool and then keeps you honest with role-play and recording analysis.

## Why it exists

Most "AI SDR" tools promise to do outbound for you. They mostly produce template-grade slop that gets ignored. This plugin produces something different: a research-grounded playbook the human owns and runs. Six rules:

1. **Every script line cites a framework** — Jeb Blount's 5-step phone, Chris Voss's mirroring/labeling, Phil M. Jones's magic words, Josh Braun's "Poke the Bear", Aaron Ross's predictable revenue, Neil Rackham's SPIN diagnostic questions. No improvisation, no template fluff.
2. **Empirical research first** — Gong's 100k-call analysis (talk:listen 55:45, opener patterns, optimal pace, filler word impact), Huthwaite's 35,000-call SPIN study, Cialdini's replicated triggers. These are evidence, not motivational packaging.
3. **The role-play mode is load-bearing** — Claude plays a skeptical prospect from a persona library, the human practices the script live, the conversation is scored against the chosen framework. Every script gets reps before it touches a real prospect.
4. **The recording loop closes the gap** — drop a call recording, the plugin transcribes via Deepgram/Azure Speech and runs Gong-style analysis (talk ratios, monologue duration, objection patterns, successful closes), then recommends script updates. Solo founders get measurable iteration.
5. **Framework-tier discipline** — every cited reference in `knowledge/` is labelled `tier: evidence` (load-bearing — Gong empirical findings, Huthwaite SPIN study, Cialdini triggers, Voss negotiation provenance), `tier: scaffolding` (forcing function — Fanatical Prospecting 30-Day Rule, SPIN questioning structure, Outreach/SalesLoft cadence patterns), or `tier: vocabulary-only` (Challenger 5 seller profiles, Sandler cult patterns, most positioning mad-libs). Agents carry tiers forward when they cite, so output makes the basis of every recommendation legible.
6. **State lives in the target project, not the plugin** — same composition model as `brand-plugin` and `greenfield-plugin`. `.sales/` in the project root. Reentrant skills.

## The 6 agents

| Agent | Owns | Produces |
|---|---|---|
| **sales-researcher** | Refreshes the outbound research landscape | `research/state-of-outbound.md`, `research/empirical-findings.md`, `research/category-context.md` |
| **icp-analyst** | ICP definition, list sourcing, dedupe and scoring | `icp.md`, `lists/strategy.md`, scrape recipes |
| **script-writer** | Cold-call scripts, cold-email templates, objection libraries | `scripts/cold-call-v1.md`, `scripts/cold-email-sequence.md`, `scripts/objections-library.md` |
| **cadence-designer** | Multi-touch sequences (call → email → LinkedIn → retry) | `cadences/<name>.md` |
| **crm-architect** | HubSpot Free / Pipedrive / Close.io setup with API patterns | `crm-setup.md`, optional `crm-bootstrap.json` |
| **call-coach** | Recording transcription + Gong-style analysis + script update recommendations | `recordings-analysis/<recording-id>.md` |

## Workflow

```
empty dir
   ↓
/sales:init        →  interview, define ICP, capture constraints
/sales:research    →  sales-researcher refreshes the empirical landscape
/sales:list        →  icp-analyst produces list-sourcing strategy + scoring rules
/sales:script      →  script-writer produces scripts + objection library
/sales:cadence     →  cadence-designer produces multi-touch sequence
/sales:crm-setup   →  crm-architect produces CRM setup guide (+ optional API automation)
/sales:role-play   →  Claude plays a prospect persona, user practices, scored against chosen framework
   ↓
   user runs real cold calls, records them
   ↓
/sales:coach       →  call-coach ingests recording, transcribes, analyzes, recommends script updates
/sales:status      →  state reporter (any time, reentrant)
```

Active plugin time per session: ~60-120 minutes for the full pipeline. After that, the role-play and coach loops run continuously as the user iterates from real call recordings.

## Composition with sibling plugins

- **`brand`** — reads `.brand/voice.md` so cold-call scripts and emails sound like the brand. The voice signature carries directly into `scripts/cold-call-v1.md`.
- **`marketing`** — receives sales closed/lost feedback for attribution; pulls inbound waitlist leads from `.marketing/leads/` into outbound sequences.
- **`greenfield`** — reads `.greenfield/data-model.md` to know what fields exist in the contractor table for list building (relevant for marketplace projects like elocal_clone).

This plugin reads from `.brand/` and `.marketing/` if present, never writes outside `.sales/`.

## State directory

All output lives in `.sales/` in the target project, never in the plugin directory. The full tree is documented in `docs/workflow.md`. Highlights:

- `icp.md` — locked ICP definition
- `lists/` — scraping strategy, sourcing recipes, scoring rules
- `scripts/` — cold-call and cold-email scripts with cited objection responses
- `cadences/` — multi-touch sequences
- `crm-setup.md` — HubSpot / Pipedrive / Close.io setup guide
- `role-play-transcripts/` — practice session transcripts with framework scoring
- `recordings-analysis/` — coach analysis of real calls

## Status

**v0.1.0** — first release. Six agents, nine skills, knowledge bundle with framework extracts and tier annotations, role-play persona library, recording-analysis loop wired for Deepgram (default) and Azure Speech (for the elocal_clone all-Azure stack). Iterate from real use on `elocal_clone`.

## License

MIT
