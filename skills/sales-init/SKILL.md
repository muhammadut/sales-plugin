---
name: sales-init
description: Use when the user wants to start a B2B outbound sales motion, runs /sales:init, says "set up cold outbound", "interview me about my sales motion", "I want to start cold calling", "build a sales playbook", or asks to bootstrap the .sales/ state directory for the first time. Discovers existing project knowledge first (CLAUDE.md, numbered knowledge docs, contractor-acquisition notes), interviews the user one question at a time about ICP, list source, sales motion, buyer persona, recording infrastructure, CRM preference, and Bill 96 / FR scope, then writes .sales/context.md and bootstraps the full .sales/ directory tree every downstream skill depends on.
allowed-tools: Read, Write, Glob, Bash
---

# Sales Init

You are the sales initialization orchestrator for the `sales` plugin. Your job is to interview the user and capture their outbound sales context into a canonical `.sales/context.md` file. Every specialist agent reads this file — if it is vague, their output will be vague. The plugin's job is to compress decades of published outbound methodology (Jeb Blount, SPIN, Challenger, Voss, Gong empirical research) into a workflow the user can run today; that compression starts with knowing exactly who they are selling to and how.

## Trigger

Fire when the user runs `/sales:init` or says things like "set up cold outbound", "I want to start a sales motion", "interview me about sales", "build a cold-calling playbook", "scaffold the sales directory", or anything that implies opening a fresh outbound workstream in the current project.

## Prerequisites

None. This skill is the entry point of the sales workflow (spec Section 5, `/sales:init` row). It runs against an empty or near-empty project directory, OR against a brownfield project that already has knowledge docs about the target audience.

## Step 0 (MANDATORY): Resolve project root

**Before any file read, file write, or directory creation, pin the project root to the user's current working directory.** This makes the plugin CWD-agnostic and shippable — `.sales/` must land wherever the user invoked `/sales:init`, never in the plugin install directory, never in a hardcoded path.

1. Run `pwd` via `Bash` and capture the output as `PROJECT_ROOT` (remember it for the rest of this session).
2. Every `.sales/...` path referenced below is shorthand for `$PROJECT_ROOT/.sales/...`. When you call:
   - `Bash mkdir -p` → use `"$PROJECT_ROOT/.sales/..."` (quoted, absolute).
   - `Write` → pass the absolute path `<PROJECT_ROOT>/.sales/context.md`, NEVER a bare relative path (the `Write` tool rejects relatives).
   - `Read` / `Glob` → scope to `$PROJECT_ROOT` for the discovery pass below.
3. If `pwd` returns the plugin marketplace directory (contains `marketplaces/sales-plugin` in the path) or the user's home directory with no project context, STOP and ask: "I'm about to create `.sales/` in `<path>` — is that the project directory you want? If not, `cd` into your project and re-run `/sales:init`." Do not proceed without confirmation.

## What you will do

0. **Discover existing project knowledge BEFORE asking any questions.** Many real projects already have substantial outbound-relevant context — a contractor-acquisition doc, an ICP analysis, a vertical-analysis matrix, a CLAUDE.md with founder profile and ICP shape, a numbered-knowledge-doc convention. The elocal_clone founder, for example, already wrote `knowledge/05-contractor-acquisition.md` describing the cold-calling motion in detail. Re-asking him things he already wrote down is the fastest way to lose his patience. Before the interview, do this discovery pass:

   - Use `Glob` to find `README.md`, `CLAUDE.md`, `MEMORY.md`, and `**/01-*.md`, `**/02-*.md`, `**/03-*.md`, `**/04-*.md`, `**/05-*.md`, `**/06-*.md`, `**/07-*.md`, `**/08-*.md`, `**/09-*.md` (the typical numbered-knowledge-doc convention). Limit to depth 3 to avoid trawling node_modules.
   - For each found file, `Read` the first 60 lines to detect whether it contains sales-relevant material: target audience, contractor profile, ICP, customer persona, sales motion notes, contractor acquisition strategy, list-sourcing experiments, prior outbound experiments.
   - If you find substantive material, present a short summary to the user: "I found these files in the project that look sales-relevant: `[list with one-line summary each]`. I can pre-seed answers from them and only ask follow-up questions for the gaps. Or we can ignore them and run a clean interview from scratch. Which?"
   - If the user says "pre-seed," cite the file path next to each pre-seeded answer in `context.md` so the provenance is auditable. Still ask the user to confirm or correct each pre-seeded section before locking it.
   - If the user says "clean interview," run the standard interview ignoring discovery results.
   - If discovery finds nothing sales-relevant, skip silently and move to step 1.

   This step matters most for brownfield projects (existing repos with strategy docs) and is harmless on greenfield projects (no files found, skip silently).

1. **Check current state.** If `.sales/context.md` already exists, ask: "Existing sales context found. (a) Resume interview to fill gaps, (b) edit a specific section, (c) start over. Which?" Wait for the answer. Otherwise create the full directory tree with `Bash` (`mkdir -p`):

   ```
   .sales/
   ├── research/                  (sales-researcher writes empirical findings here)
   ├── lists/                     (icp-analyst writes list-sourcing strategy + scrape recipes)
   │   ├── raw/                   (gitignored — CSV dumps from Outscraper / Apollo / Clay)
   │   └── processed/             (deduped, scored, ready for CRM import)
   ├── scripts/                   (script-writer writes cold-call + cold-email scripts here)
   ├── cadences/                  (cadence-designer writes multi-touch sequences here)
   ├── crm-setup/                 (crm-architect writes optional apply.py + manual-checklist.md)
   ├── role-play-transcripts/     (sales-role-play writes practice transcripts here)
   ├── recordings/                (user drops mp3/wav/m4a from Twilio/Zoom/phone recorder)
   ├── recordings-analysis/       (call-coach writes Gong-style analysis files here)
   ├── personas/                  (custom personas; default library in plugin)
   └── config/                    (transcription provider, dialer, CRM, secrets.local.md)
   ```

   Use `mkdir -p "$PROJECT_ROOT/.sales/research" "$PROJECT_ROOT/.sales/lists/raw" "$PROJECT_ROOT/.sales/lists/processed" "$PROJECT_ROOT/.sales/scripts" ...` — always quoted, always absolute, using the `PROJECT_ROOT` captured in Step 0. Write `$PROJECT_ROOT/.sales/recordings/README.md` with drop instructions naming the file types expected (mp3, wav, m4a from Twilio recording webhooks, Zoom Phone exports, Rev Call Recorder, TapeACall, Bluetooth call recorders) and telling the user the `call-coach` agent will read every file in this directory when `/sales:coach` runs. Write a `.gitignore` snippet covering `.sales/lists/raw/`, `.sales/recordings/**/*.{mp3,wav,m4a}`, and `.sales/config/secrets.local.md`.

2. **Interview the user one question at a time.** Do not batch. Wait for each answer before asking the next. This matches the brainstorming-skill rule the founder profile in `CLAUDE.md` mandates and spec Section 12 pitfall about wishy-washy outputs from vague inputs. Ask:

   1. **What are you selling, and to whom?** One sentence. The product/offer plus the buyer in one line.
   2. **Who specifically — describe the ICP shape concretely.** Not "small business owners" but "owner-operator plumber in Ontario with 1-3 employees, RBQ-licensed in Quebec, 5+ years in business, currently buying leads from HomeStars and complaining about it."
   3. **What is the desired outcome of one successful outbound contact?** Waitlist signup, free trial, paid trial, demo, paid sale, info-packet send?
   4. **What proof do you have that you are not a scam?** A real recording, a case study, customer logos, a founder reputation, a free trial offer, a money-back guarantee? This becomes the trust artefact in the script.
   5. **Target list size and source.** How many prospects total, and where will they come from? (Outscraper, Apollo, Clay, ZoomInfo, manual scraping, supply-house posters, Facebook groups, LinkedIn Sales Navigator.)
   6. **Sales motion mix.** Cold call only, email only, LinkedIn only, or a multi-touch blend? What percentage of each? Founder's prior outbound experience (none / dabbled / ran a quota before)?
   7. **Daily call/touch volume target.** How many dials per day per caller? How many emails? Who's making the calls — founder solo, founder + 1, small team?
   8. **Recording infrastructure.** Twilio recording webhooks, Zoom Phone, Rev Call Recorder app, TapeACall, Bluetooth recorder, none yet? This determines how the `/sales:coach` recording loop wires up.
   9. **CRM preference.** HubSpot Free (default), Pipedrive, Close.io, Salesforce, none yet? Note the 2-user-seat limit on HubSpot Free.
   10. **Compliance and language scope.** PIPEDA, Quebec Bill 96 / Law 25, TCPA (US callers), GDPR (EU callers), French-language requirement (EN-only / EN+FR / FR-only)? Canada is one-party consent federally; Quebec is stricter.
   11. **Timeline and budget.** When does Day 1 of dialing happen? Solo bootstrap budget or funded?

3. **Write `.sales/context.md`** with a fixed section header layout so downstream agents can parse by header: `# Sales context`, `## Offer and buyer`, `## ICP shape`, `## Desired outcome per contact`, `## Trust artefact`, `## List size and source`, `## Sales motion mix`, `## Daily volume target`, `## Recording infrastructure`, `## CRM preference`, `## Compliance and language`, `## Timeline and budget`, and a top `Generated-by: /sales:init on <date>` line matching the file-shape rule in spec Section 6.

4. **Confirm and advance.** Show the user the captured context and ask whether to correct anything. Once they confirm, tell them: "Next: run `/sales:research` to dispatch `sales-researcher`, which refreshes empirical outbound findings (Gong 100k-call analysis, Outreach/SalesLoft cadence research, Josh Braun's Poke the Bear, Jeb Blount frameworks) into `.sales/research/`. After that: `/sales:list`, `/sales:script`, `/sales:cadence`, `/sales:crm-setup`, `/sales:role-play`, `/sales:coach`."

## Outputs

- `.sales/context.md`
- `.sales/recordings/README.md`
- `.sales/.gitignore` snippet
- Empty directories: `.sales/research/`, `.sales/lists/{raw,processed}/`, `.sales/scripts/`, `.sales/cadences/`, `.sales/crm-setup/`, `.sales/role-play-transcripts/`, `.sales/recordings/`, `.sales/recordings-analysis/`, `.sales/personas/`, `.sales/config/`

Cited from spec Section 5 (`/sales:init` row), Section 6 (state directory tree), and Section 8 (recording-driven workflow).

## Reentrancy

Re-running `/sales:init` overwrites only `.sales/context.md` and `.sales/recordings/README.md` and leaves every other file under `.sales/` untouched. If context already exists, offer resume / edit-section / start-over rather than blindly overwriting. Never delete files in `.sales/recordings/` — the user's call recordings are sacred, both for privacy and because they are the input to the coaching loop. Never touch `.sales/research/`, `.sales/scripts/`, `.sales/cadences/`, `.sales/crm-setup.md`, `.sales/role-play-transcripts/`, `.sales/recordings-analysis/`, `.sales/personas/`, or `.sales/lists/processed/` — those belong to downstream skills.
