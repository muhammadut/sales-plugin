# Agent Roster & Handoff Protocol

## The 6 agents (7 mode-rows because call-coach has two)

Each agent produces one or more artefacts in `.sales/`. No agent reads its own output. The `call-coach` is the only agent that operates in two modes from a single file — both modes share a single agent definition (`agents/call-coach.md`) and dispatch via different skills (`/sales:coach` for coach mode, `/sales:role-play` for role-play mode). It is shown as two rows below for clarity.

| Agent | Mode | Primary input | Primary output | Model |
|---|---|---|---|---|
| `sales-researcher` | single | `.sales/context.md`, `${CLAUDE_PLUGIN_ROOT}/knowledge/frameworks/*`, `${CLAUDE_PLUGIN_ROOT}/knowledge/reports/*`, live web | `.sales/research/state-of-outbound.md`, `.sales/research/empirical-findings.md`, `.sales/research/category-context.md` | sonnet |
| `icp-analyst` | single | `.sales/context.md`, `.sales/research/*.md`, `.greenfield/data-model.md` (optional) | `.sales/icp.md`, `.sales/lists/strategy.md`, `.sales/lists/scoring-rules.md` | opus |
| `script-writer` | single | `.sales/context.md`, `.sales/icp.md`, `.sales/research/*.md`, `.brand/voice.md` (optional), `${CLAUDE_PLUGIN_ROOT}/knowledge/frameworks/*` | `.sales/scripts/cold-call-v<n>.md`, `.sales/scripts/cold-email-sequence.md`, `.sales/scripts/objections-library.md` | opus |
| `cadence-designer` | single | `.sales/icp.md`, `.sales/scripts/*`, `.sales/research/*.md`, `${CLAUDE_PLUGIN_ROOT}/knowledge/frameworks/*`, `${CLAUDE_PLUGIN_ROOT}/knowledge/reports/*` | `.sales/cadences/<name>.md` | sonnet |
| `crm-architect` | single | `.sales/icp.md`, `.sales/cadences/*` | `.sales/crm-setup.md`, optional `.sales/crm-bootstrap.json` | sonnet |
| `call-coach` | coach | `.sales/recordings/<date>/*.{mp3,wav,m4a}` (or transcript path argument), `.sales/scripts/*`, `${CLAUDE_PLUGIN_ROOT}/knowledge/frameworks/*`, `${CLAUDE_PLUGIN_ROOT}/knowledge/reports/*` | `.sales/recordings-analysis/<recording-id>.md` | opus |
| `call-coach` | role-play | `.sales/scripts/cold-call-v<n>.md`, `.sales/icp.md`, `${CLAUDE_PLUGIN_ROOT}/personas/<persona>.md` (or `.sales/personas/`), live user input | `.sales/role-play-transcripts/<session-id>.md` | opus |

`sales-status` is **a skill, not an agent** — it reads the `.sales/` tree and prints a console report without dispatching any agent. See `skills/sales-status/SKILL.md`. Skills under `skills/` are the user-facing entry points; they invoke the agents above. See `docs/workflow.md` for the phase-by-phase dispatch table.

## Handoff protocol

Agents never talk to each other directly. All communication is via files in `.sales/`. Each agent:

1. Reads its declared inputs (table above).
2. Writes its declared output.
3. Emits a short (<300 word) summary back to the orchestrating skill.

Every output file has a fixed header block with three fields — `Generated-by:`, `Depends-on:`, `Sources-cited:`. The `sales-status` skill validates these on every run. A file without the header is a bug; a file whose `Depends-on:` points at a missing input is a bug; a file whose `Sources-cited:` is empty when the agent's contract requires citations is a bug.

Why file-based, not message-based:

- **Context isolation.** Each agent has minimal context; no token burn on other agents' deliberations.
- **Debuggability.** The user inspects what each agent actually produced — no opaque in-memory graph.
- **Resumability.** Partial workflows resume by re-running specific skills against the existing state.
- **Git-committable.** The filesystem is the repo's source of truth, not a live message bus.
- **Audio + transcript feeds.** The `call-coach` in coach mode reads `.mp3`/`.wav` paths and their generated `.transcript.txt` sidecars directly from `recordings/`. Files-as-inputs is the only shape that accommodates audio reads naturally.

## `call-coach` two-mode operation

The `call-coach` is the only agent in the roster that operates in two distinct modes, and it is worth explaining why they are collapsed into one agent rather than split into two.

**Coach mode** (invoked during `/sales:coach`): read every new recording in `.sales/recordings/<date>/`, ensure each has a `.transcript.txt` sidecar (transcribe via Deepgram, AssemblyAI, Azure Speech, or local Whisper per `.sales/config/config.yaml`), then read the active script and the framework knowledge bundle. Score the call on Gong-style metrics — talk-to-listen ratio (target 55:45 for cold calls, `tier: evidence`), monologue duration (target ~53s, `tier: evidence`), opener detection against the script, objection classification against the RBO categories, filler-word density, outcome label. Write `.sales/recordings-analysis/<date>.md`. Aggregate into `.sales/coach-notes.md`. When the same un-handled objection appears 3+ times, propose `.sales/scripts/call-script-v<n+1>.md` and surface a diff. Output template: `templates/recording-analysis-template.md`.

**Role-play mode** (invoked during `/sales:role-play`): load a persona from `${CLAUDE_PLUGIN_ROOT}/personas/<persona>.md` (or `.sales/personas/<persona>.md`), commit to the persona — vocabulary, objection patterns, decision-making style, refusal to break character — and exchange messages with the user. Track live: did the user open per the script? did they hit the value statement? did they handle the first objection? did they ask for the close? When the conversation ends (sign-up, hang-up, or user gives up), produce a scorecard against the chosen framework with checkmarks for hit steps and Xs for missed steps, talk-to-listen ratio (estimated from word counts), filler word count, 3-5 framework-cited coaching notes, and a suggested next scenario. Write `.sales/role-play-transcripts/<date>-<scenario>.md`. Persona template: `templates/persona-template.md`.

These could be two separate agents — `recording-coach` and `role-play-partner`. They were collapsed into one for three reasons:

1. **Identical scoring engine.** The same framework-citation discipline that scores a real recording also scores a simulated conversation. Splitting them would require duplicating the scorecard rubric, the RBO objection classifier, the Gong-metric definitions, and the citation tier-tracking in two system prompts, risking drift.
2. **Shared knowledge load.** Both modes read the entire `${CLAUDE_PLUGIN_ROOT}/knowledge/` bundle, the active script, and the objection library. Two agents reading 95% the same context is exactly the duplication brand-plugin's add/remove rule warns against.
3. **Hand-off cleanliness.** A workflow that ran user → recording-coach → user → role-play-partner → user → recording-coach v2 would push context back and forth unnecessarily. One agent with two modes lets the system prompt reason about "the user practiced this scenario yesterday and missed the objection bridge — surface it again in coach mode if I see it in a real recording."

The system prompt branches on the invoking command. `/sales:coach` invokes coach mode; `/sales:role-play` invokes role-play mode. See `agents/call-coach.md` for the branching language.

## When to add an agent

Borrowed from greenfield's and brand's discipline. **Add an agent** if, and only if:

- A new deliverable is genuinely needed that doesn't fit any existing agent's charter; AND
- It has its own inputs, outputs, and tools; AND
- It represents an independent expertise that would not just re-run the same reasoning on the same files.

**Do not add** an agent if:

- The "role" is a re-framing of an existing one. The earlier draft's proposed `role-play-partner` collapsed into `call-coach`'s second mode for exactly this reason.
- The role exists only during downstream phases (e.g., enterprise account management). Push it to the right plugin (`accounts`, `revops`).
- Its outputs would always be read alongside, and identical-in-reasoning-to, an existing agent's outputs.

## When to remove an agent

**Remove** if:

- Two agents end up reading the same inputs and writing overlapping outputs. Collapse them.
- An agent's output is consistently empty or trivial for realistic project types.
- An agent's output is always subsumed by another agent's output.

The six is a default, not a mandate. For a founder who already has a CRM and never wants the plugin to touch it, `crm-architect` may be skipped and the phase collapsed into a hand-off note. For a project that is email-only with no phone outreach, `call-coach`'s coach mode never runs and only role-play mode is exercised.

## Model assignment rationale

Three agents are assigned `opus`, three are assigned `sonnet`. The split tracks reasoning depth, not output length.

- **`script-writer` → opus.** This is the highest-leverage reasoning in the pipeline. Picking the right framework for the ICP, writing lines that survive contact with a skeptical contractor, mapping every objection in the library to a framework-cited response — this is dense linguistic + strategic work. Getting the script wrong poisons every dial. Opus-class reasoning is worth the cost.
- **`icp-analyst` → opus.** Defining the ICP firmographically, picking inclusion/exclusion criteria that survive scraping reality, building the disqualification heuristics that incorporate the HomeAdvisor-reverse-playbook items — this requires synthesizing research, sibling-plugin context, and competitive intelligence. Opus earns its keep.
- **`call-coach` → opus.** Both modes apply a multi-axis rubric to conversation transcripts with cited reasoning, then propose actionable changes. Coach mode in particular runs LLM classification of objections against the RBO categories and diffs transcripts against the script — this is exactly the kind of work where the opus reasoning gap shows up. Role-play mode also benefits because persona fidelity (refusing to drift into generic helpfulness) is a known opus strength.
- **`sales-researcher` → sonnet.** The job is structured research and synthesis — fetch, summarize, organize into one file. No deep reasoning; the depth comes from the live sources the agent cites. Sonnet is sufficient and the cost savings matter because researcher runs first and refreshes regularly.
- **`cadence-designer` → sonnet.** Structural: 8-16 touches across 21+ days, channel mix per Outreach/SalesLoft research, content reference per touch. Creative but shaped. Sonnet handles this cleanly.
- **`crm-architect` → sonnet.** Structural work — pipeline stages with exit criteria, custom fields with type and purpose, KPI dashboard math, optional API-bootstrap script. The strategic decisions (which CRM, which fields) are bounded by the ICP; the agent's job is to format and validate. Sonnet with a strict prompt produces more consistent schema-compliance than opus's occasional cleverness.

Model assignments are declared in each agent's YAML frontmatter (`model: opus` or `model: sonnet`) and in `.claude-plugin/plugin.json`. Override per project via environment if cost is an issue — script-writer and icp-analyst in particular can fall back to sonnet for prototype runs, but the production run should use opus.
