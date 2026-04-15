# Sales Plugin — Design Document

> v0.1.0 — synthesized from `plugin-specs/sales-plugin-spec.md` and pattern-matched against `brand-plugin/docs/` and `greenfield-plugin/docs/`.

## Purpose

`sales` is a Claude Code plugin for the cold-outbound phase of a B2B motion. It interviews the founder, dispatches six specialist agents, and produces a `.sales/` directory containing every artefact a two-person team needs to start cold-calling and emailing on day one — ICP, list-sourcing strategy, scripts that cite named frameworks, multi-touch cadences, CRM setup, role-play practice transcripts, and a recording-driven coaching loop. It is for solo founders and small teams running outbound for the first time, who need a defensible playbook they can run today and then iterate from real call recordings tomorrow.

**What it explicitly does NOT do:** inbound sales (chat widgets, demo bookings, MQL routing — the marketing plugin owns inbound), enterprise account management (QBRs, multi-thread mapping, mutual action plans), generic sales training or certification, RevOps / commission planning / forecasting, channel sales, or — critically — replacing the human caller with an AI SDR. The plugin trains the human and instruments the human; it does not pretend the AI is doing the selling. That lie would break the loop because the founder would stop learning and the playbook would never improve.

## The recording-driven coaching architecture

This is the load-bearing architectural choice and the one most worth defending.

The obvious design is "buy Gong." Gong (and Chorus, ExecVision, Salesforce Einstein Conversation Insights) charges $1,500-2,500 per user per year for a conversation-intelligence stack that records calls, transcribes them, scores them against a methodology, and recommends improvements. For a two-person team that is $3,000-5,000/year minimum — and the elocal_clone budget is $1,000/month TOTAL across infrastructure, ad spend, contractor outreach, and tooling. Gong is out.

The plugin replicates the core loop for the cost of a Deepgram subscription (~$5-20/month). The user records their cold calls — Twilio webhook, Zoom phone export, a phone recorder app, the elocal_clone routing engine, whatever the source — and drops the audio into `.sales/recordings/<date>/`. The user runs `/sales:coach` (or passes a path to `/sales:coach <file>`). The skill transcribes via Deepgram (default) or Azure Speech (when the greenfield Azure stack is detected), then dispatches the `call-coach` agent with the transcript, the active script, and the bundled framework knowledge. The agent runs Gong-style analysis — talk-to-listen ratio against the 55:45 cold-call benchmark, monologue duration against the ~53-second target, opener detection against the script, objection classification against Jeb Blount's RBO categories, filler-word density, outcome label — and writes `.sales/recordings-analysis/<date>.md` with a coaching note for every observation, every note citing the framework that produced it. Patterns aggregate into `.sales/coach-notes.md`. When the same un-handled objection appears 3+ times, the agent proposes `.sales/scripts/call-script-v<n+1>.md` and surfaces a diff against the current version. The user accepts, rejects, or hand-edits.

The closed loop turns a solo team into something like a Gong-instrumented sales floor for near-zero cost. It is what differentiates this plugin from generic AI-SDR tools — Apollo AI, Clay agents, Outreach Kaia — that try to replace the human with template-grade slop. Those tools concentrate failure: the human stops learning, the templates flatten, and the bot's call-graph rapidly diverges from real prospect conversations because nobody is watching the recordings. The recording loop is the antidote. The human runs the calls. The plugin watches the calls. The script gets better.

## The role-play mode

Cold calling is a craft that decays without practice. A founder who has never made a cold call cannot read a v1 script and execute it well — they sound robotic, get derailed by the first objection, and panic. The wife (who has even less sales experience in the elocal_clone case) is in the same boat.

A traditional sales floor solves this with manager-led role-play, peer coaching, and ramp-up programs. A solo founder has none of those. So the plugin ships a role-play mode: `/sales:role-play` loads a persona from `${CLAUDE_PLUGIN_ROOT}/personas/` (or a project-custom one in `.sales/personas/`), Claude commits to the persona — "Dave, 52, Hamilton plumber, 2-truck shop, burned by HomeStars, picks up because he is between calls" — and the user practices live. Claude raises realistic objections, uses contractor-appropriate vocabulary, refuses to break character. The session ends with a scorecard against the chosen framework (Fanatical Prospecting 5-step, SPIN, Challenger, Voss tactical empathy), 3-5 framework-cited coaching notes, and a transcript saved to `.sales/role-play-transcripts/`.

Every script gets reps before it touches a real prospect. The role-play feature is the difference between day-1 panic and day-1 confidence. It is also the cheapest possible insurance against burning the first 50 dials on a script the founder cannot execute.

## State ownership

All plugin output lives in `.sales/` inside the **target project**, never inside the plugin directory. Same convention as `brand` (`.brand/`) and `greenfield` (`.greenfield/`). Reasons:

- **Reentrant.** Every skill is idempotent on its inputs. Re-running a skill overwrites only its own outputs. Partial workflows resume cleanly.
- **Git-committable.** The user commits `.sales/` (selectively — markdown files yes; raw audio, scraped lists, and secrets no) and treats it as project source of truth.
- **Composable.** Sibling plugins read from `.sales/` without coupling to the plugin's internals. If the plugin is uninstalled, the project's `.sales/` directory still holds the playbook.
- **Every agent reads from explicit file paths.** Agents do not message each other. They read declared inputs and write declared outputs. The filesystem is the message bus; see `docs/agent-roster.md`.

## Framework-tier discipline

Every cited reference in `${CLAUDE_PLUGIN_ROOT}/knowledge/` is labelled with one of three tiers. This exists to prevent framework-citation theater — the habit of salting output with big-name references to look rigorous while the underlying reasoning is unchanged.

- **`tier: evidence`** — load-bearing, empirically defensible. Gong's 100k-call analysis (talk:listen 55:45, opener data, monologue duration, successful call duration). Huthwaite's 35,000-call SPIN study. Outreach.io and SalesLoft cadence research. Cialdini's replicated influence triggers. If an agent cites an `evidence` source, the claim stands or falls on the source being correct.
- **`tier: scaffolding`** — forcing functions. Jeb Blount's 5-step phone framework. Blount's 30-Day Rule and refusal-to-quit math. SPIN's question-architecture taxonomy. Voss's mirroring/labeling/calibrated-question structure. Phil M. Jones's magic-word patterns. These are disciplines an agent follows to structure its thinking; they shape the output's shape more than its content.
- **`tier: vocabulary-only`** — conceptual labels the agent uses to communicate, without treating them as predictive models. Challenger's five seller profiles. Sandler cult patterns. Most positioning mad-libs. These are useful shared vocabulary between the agent, the user, and downstream coaches, but they should not be cited as if they predict prospect behavior.

Agents carry the tier forward when they cite. A coaching note that says "your monologue average is 18s; Gong's 100k-call analysis (tier: evidence) shows successful cold calls average 53s" is honest about what work the citation is doing. A script section that says "use a Voss-style label here (tier: scaffolding)" is honest that this is a structural recommendation, not an empirical claim.

See `${CLAUDE_PLUGIN_ROOT}/knowledge/README.md` for the full tier catalog.

## Composition with sibling plugins

`sales` is a midstream node. It runs after `brand` and reads from `marketing` and `greenfield` when they exist. It never writes outside `.sales/`.

- **`brand`** — reads `.brand/voice.md`. The script-writer agent uses the voice signature so cold-call scripts and emails sound like the brand. If absent, script-writer asks two calibrating tone questions and proceeds.
- **`greenfield`** — reads `.greenfield/data-model.md` (or `.greenfield/plan/data-architecture.md`). The icp-analyst checks what fields exist in the contractor table downstream so the CRM custom fields align with the production schema. Critical for marketplace projects like elocal_clone.
- **`marketing`** — reads `.marketing/leads/inbound/*.csv`. Inbound waitlist leads from marketing campaigns get routed into the cadence as a separate "warm follow-up" sequence.
- **`marketing`** — writes `.sales/closed-lost-reasons.md` periodically. The marketing plugin reads this for attribution. This is the only "outbound" composition arrow in the design and it is read-only on the marketing side: sales emits a summary, marketing consumes it.

The rule is unidirectional and hard: no plugin ever writes outside its own state directory. Violations break the composition contract.

## Why six agents, not nine

Greenfield runs nine because architectural decisions compound across many independent specialties — product, security, data, cloud, platform, compliance. Outbound sales has fewer truly independent specialties. List, script, cadence, and CRM are sequential phases of a single motion, not parallel domains the way "platform engineering" and "data architecture" are parallel inside greenfield. Collapsing them into 4 builder-agents + 1 researcher + 1 coach is the right shape:

1. `sales-researcher` — refreshes the published outbound research landscape (Gong, Outreach, SalesLoft, LinkedIn, Salesforce, Josh Braun)
2. `icp-analyst` — defines the ICP, list-sourcing strategy, scoring rules, disqualification heuristics
3. `script-writer` — picks one framework per script and writes the call script, objection library, talk tracks
4. `cadence-designer` — multi-touch sequences with timing, channel mix, content per touch
5. `crm-architect` — pipeline shape, custom fields, lifecycle stages, KPI dashboard, optional API bootstrap
6. `call-coach` — recording transcription, Gong-style analysis, role-play scoring, script update proposals (two modes — see `docs/agent-roster.md`)

The two roles that look like they could collapse but should NOT:

- **`sales-researcher` vs `script-writer`** — separated because research must be neutral and citation-heavy, while script-writer must commit to a single framework per script. Mixing them produces wishy-washy scripts that cite everything and recommend nothing.
- **`script-writer` vs `call-coach`** — separated because script-writer is generative (works from frameworks) while call-coach is observational (works from transcripts). Different epistemic stance, like the AgentCoder argument for separating test design from code generation.

An earlier draft considered separate `role-play-coach` and `recording-coach` agents. Both consume conversation transcripts (one simulated, one real), both score against frameworks, both output framework-cited reasoning. Splitting them would duplicate the entire framework-citation engine. They collapsed into `call-coach` with two modes — same shape as brand-plugin's `creative-director`. See `docs/agent-roster.md`.
