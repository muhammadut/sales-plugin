# Known Gaps — sales-plugin v0.1.0

A running list of known limitations, deferred features, and reviewer-flagged items the v0.1.0 ship intentionally does not address. Update this file every release; do not silently fix items here without updating the spec or the release notes.

## Architectural by-design (not bugs)

These are *features* of the v0.1.0 architecture, captured here so a new reader does not mistake them for oversights.

- **Recording transcription requires the user to provide an audio file path.** The plugin does not attach to phone systems, dialers, or PBXs directly. Users record calls on their preferred system (Twilio webhook, Zoom export, phone recorder app, elocal_clone routing engine), drop the file into `.sales/recordings/<date>/`, and run `/sales:coach`. Reasons: tool diversity (no two users share a dialer), cost control (no plugin-held subscriptions to recording vendors), privacy (the user owns the audio and decides when it gets transcribed). See `docs/design.md`.
- **The plugin does not replace the human caller.** Explicit anti-goal. Generic AI-SDR tools (Apollo AI, Clay agents, Outreach Kaia) try to replace the human and produce template-grade slop. This plugin trains the human and instruments the human. The human still dials.
- **No GUI / dashboard.** The plugin is CLI-only by design. Users who want a visual board look at the `.sales/` directory tree directly, run `/sales:status`, or commit `.sales/` to a git repo and view it in the GitHub web UI.
- **No plugin-held API keys.** The plugin never holds Deepgram, Azure Speech, AssemblyAI, HubSpot, Pipedrive, or any other provider's keys. Users supply their own via `.sales/config/secrets.local.md` (gitignored). Out of scope permanently.
- **EN-primary outbound by default.** v0.1 ships with English scripts and personas. Quebec French support is a `--bilingual` flag deferred to v0.2 — except the persona library does ship one bilingual Quebec contractor archetype so the role-play mode can practice code-switching.

## v0.2 deferrals

Features the v0.1 build deliberately stubbed or skipped, scheduled for the v0.2 sweep:

- **HubSpot Free is the v0.1 default; Pipedrive and Close.io alternatives are documented but not API-bootstrapped.** `crm-architect` writes a Pipedrive-flavored or Close.io-flavored `.sales/crm-setup.md` when those CRMs are picked, but only HubSpot has a working `crm-setup/apply.py` API bootstrap in v0.1. v0.2 wires the API bootstrap for all three.
- **Deepgram is the default transcription provider; Azure Speech is available when the greenfield Azure stack is detected.** AssemblyAI and local Whisper are documented in `.sales/config/config.yaml` and supported by the call-coach agent reading the config, but they are not auto-detected. v0.2 should auto-pick AssemblyAI when sentiment/summarization is requested and surface a privacy-mode toggle for local Whisper.
- **Persona library starts with 3 personas in v0.1.** `skeptical-hamilton-plumber`, `defensive-electrician`, `busy-hvac-owner` ship with v0.1. The full 8-12 archetypes (price-shopper, time-pressed, friendly stall, hostile, polite-stall, interested-but-confused, HomeStars-burned, bilingual Quebec contractor, gatekeeper, competitor's customer) are scheduled for v0.2.
- **Voice-mode role-play.** v0.1 ships text-mode role-play only — the user types responses to Claude's persona messages. Voice mode (user speaks, Claude transcribes live and responds) is deferred to v0.2 and depends on Deepgram or Azure Speech streaming APIs.
- **Bilingual mode (full Quebec French).** Every agent producing EN and FR variants in parallel is opt-in via a `--bilingual` flag deferred to v0.2. v0.1 supports French only via the bundled Quebec persona for role-play and via a script-writer flag that adds a "French equivalent" appendix when the ICP includes Quebec.
- **`/sales:plan-all` mega-command.** A future command that runs research → list → script → cadence → crm-setup in parallel with a reconciliation step at the end. v0.1 is sequential (one command per agent) by design, to keep the dispatch model debuggable.
- **Migration story for projects upgrading from v0.1 to v0.2.** Not designed yet. Document on v0.2 release.

## Phase 4 fixes already applied

These were flagged by the spec-compliance reviewer and the plugin-validator in Phase 3 and fixed before this v0.1.0 ship — listed here so the audit trail is complete:

- **Knowledge directory restructured.** Build agents created `knowledge/methodologies/`, `knowledge/cold-email/`, `knowledge/objection-handling/`, `knowledge/empirical-research/`, `knowledge/blue-collar/`, `knowledge/crm-tooling/`, but every agent and skill referenced `knowledge/frameworks/` and `knowledge/reports/`. Phase 4 flattened the build into the two-directory structure the consumers expect (16 framework extracts in `knowledge/frameworks/`, 2 empirical reports in `knowledge/reports/`). `knowledge/README.md` was rewritten to reflect the new structure.
- **Personas directory created.** Build agents referenced `${CLAUDE_PLUGIN_ROOT}/personas/` in 9+ files (especially `agents/call-coach.md` line 66 hard-coding `hamilton-plumber.md` as the default for `/sales:role-play`) but the directory didn't exist. Phase 4 created `sales-plugin/personas/hamilton-plumber.md` — a fully-fleshed canonical persona for the elocal_clone first user with identity, current pain, prior burns (HomeAdvisor / Angi / Networx / two anonymous SaaS reps), what works on him (recording-as-proof, founding-member framing, naming a peer), what turns him off, 8 verbatim objections, words he uses, words he never uses, and a defining quote. The remaining 11 personas advertised in `sales-role-play/SKILL.md` are deferred to v0.1.1.
- **`docs/agent-roster.md` rewritten** to match the actual artefact paths the skills and agents produce (`.sales/research/state-of-outbound.md` not `.sales/research.md`, `.sales/scripts/cold-call-v<n>.md` not `.sales/scripts/call-script-v<n>.md`, `.sales/scripts/objections-library.md` not singular `objection-library.md`, `.sales/cadences/<name>.md` not `.sales/cadences/primary.md`, `.sales/recordings-analysis/<recording-id>.md` not `<date>.md`).
- **`templates/cold-call-script-template.md` Depends-on header fixed** to reference `.sales/research/state-of-outbound.md` and `.sales/research/empirical-findings.md` instead of the non-existent `.sales/research.md` single file.
- **`.gitignore` repaired.** The plugin's own `.gitignore` had an unqualified `.sales/` line which would have suppressed legitimate state-directory artefacts in any target project that co-located with the plugin source. Phase 4 removed the line and added a comment documenting the spec's selective-ignore convention for target projects (only `lists/raw/`, `recordings/**/*.{mp3,wav,m4a}`, `config/secrets.local.md` should be ignored; markdown artefacts SHOULD be committed).

## Reviewer-flagged polish (deferred to v0.1.1)

These were flagged by Phase 3 reviewers but accepted as-is for v0.1.0 because they are stylistic, cosmetic, or non-blocking. v0.1.1 sweep should clean them up.

- **Persona library coverage.** Only `hamilton-plumber.md` ships in v0.1; the role-play skill advertises 11 more (skeptical-owner-operator, franchise-manager, price-shopper, time-pressed, hostile, polite-stall, interested-but-confused, homestars-burned, quebec-bilingual, non-answerer, gatekeeper, competitors-customer). Add the remaining 11 in v0.1.1 using the persona-template and the hamilton-plumber file as the shape.
- **`kpi-dashboard-spec.md` orphan.** Section 11 success criterion #7 implies a `.sales/kpi-dashboard-spec.md` output, but the `crm-architect` agent and `sales-crm-setup` skill don't currently write it. v0.1.1 adds the file as a hard output of `crm-architect`.
- **`sales-status --strict` header audit will fail against its own spec.** The `--strict` mode requires every markdown file in `.sales/` to carry `Generated-by:` / `Depends-on:` / `Sources-cited:` headers, but the orchestrator skills don't currently force the writer agents to produce them in the dispatch prompts. v0.1.1 either softens `--strict` to warn-not-fail or adds the header template to every dispatch prompt.
- **`sales-coach` Azure Speech auto-detection.** The skill says "default to Azure Speech when `.greenfield/` indicates Azure stack" but the auto-detection logic is not wired in v0.1. Manual config flag works as a fallback. v0.1.1 wires the auto-detect.
- **`call-coach` AssemblyAI / Whisper anti-pattern over-restricts the spec.** The agent's anti-patterns block forbids AssemblyAI and local Whisper, but the spec and `.sales/config/config.yaml` allow them. v0.1.1 softens the anti-pattern to "must not call unconfigured providers" and lets the config pick.
- **README "tier count" bookkeeping** says "18 extracts + 1 README = 19" which is technically self-referential. v0.1.1 fixes the heading to "18 extracts (plus this README)".
- **Sibling-plugin path drift.** `agents/icp-analyst.md` says `.greenfield/data-model.md`; `skills/sales-list/SKILL.md` says `.greenfield/plan/data-architecture.md`. README says the former. Pick one in v0.1.1.

## Open questions for v0.1 → v0.1.1

- Should `/sales:status --strict` validate that every framework cited in `.sales/scripts/call-script-v<n>.md` resolves to a file in the bundled `${CLAUDE_PLUGIN_ROOT}/knowledge/` directory? Currently it just counts named frameworks via grep — a stronger check would catch hallucinated citations.
- Should `/sales:coach` auto-propose a script update on the first un-handled objection, or wait for the 3-occurrence threshold the spec defines? v0.1 waits for 3; revisit if users report missing the early signal.
- Should role-play sessions count toward script-version bumps the way real recordings do? v0.1 keeps them separate (role-play transcripts inform coaching notes but never trigger a new script version) on the principle that a simulated objection is not the same as a real one.

## How to update this file

When a v0.1.x or v0.2 release ships, move the resolved items to a "Resolved in v0.X.Y" section at the bottom and add new gaps as they are discovered. Do not silently delete items — the change history is load-bearing for understanding why decisions were made.
