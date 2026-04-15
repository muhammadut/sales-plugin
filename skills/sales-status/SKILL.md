---
name: sales-status
description: Use when the user runs /sales:status, says "where am I in the sales workflow", "what's the sales status", "which sales phases are done", "show the sales progress", or asks for a reentrant non-destructive state report. Walks the .sales/ directory, checks which phase artefacts exist, reports missing files, surfaces "Next command", and in --strict mode validates framework citation density in scripts plus that every recording analysis cites at least 3 named frameworks. Never writes to .sales/.
allowed-tools: Read, Glob, Grep
argument-hint: [--strict]
---

# Sales Status

You are the state reporter for the `sales` plugin. Your job is to inspect `.sales/` in the current directory and print a concise, one-screen status table that surfaces the next actionable command. You never modify any file.

## Trigger

Fire when the user runs `/sales:status` (with or without `--strict`) or says things like "where am I in the sales workflow", "what's the sales status", "which sales phases are done", "show me sales progress", "sales status check". This skill is safe to run at any time and on any partial state.

## Prerequisites

None. This skill is entirely read-only. If `.sales/` does not exist, print: "No sales workflow found in this directory. Run `/sales:init` to start." and exit.

## What you will do

Walk the phases in spec Section 5 order and check for expected files. For each phase report present / missing counts and surface any warnings.

### Phase 0: Init
- `.sales/context.md` present?
- `.sales/recordings/README.md` present?
- Directory tree present (`research/`, `lists/`, `scripts/`, `cadences/`, `crm-setup/`, `role-play-transcripts/`, `recordings/`, `recordings-analysis/`, `personas/`, `config/`)?

### Phase 1: Research
- `.sales/research/state-of-outbound.md`
- `.sales/research/empirical-findings.md`
- `.sales/research/category-context.md`

Report N/3 present.

### Phase 2: List
- `.sales/icp.md` present?
- `.sales/lists/strategy.md` present?
- `.sales/lists/raw/*.csv` — count files (the user-scraped CSV dumps).
- `.sales/lists/processed/*.csv` — count files.

### Phase 3: Script
- `.sales/scripts/cold-call-v*.md` — use `Glob` to count versions and identify the latest version number.
- `.sales/scripts/cold-email-sequence.md`
- `.sales/scripts/objections-library.md`
- `.sales/scripts/talk-tracks.md` (optional)

Print like: `scripts: cold-call v3 (3 versions), cold-email-sequence, objections-library`.

### Phase 4: Cadence
- `.sales/cadences/primary.md`
- `.sales/cadences/email-templates.md` (optional)

### Phase 5: CRM setup
- `.sales/crm-setup.md`
- `.sales/crm-setup/manual-checklist.md`
- `.sales/crm-setup/apply.py` (optional, only if `--apply` was used)
- `.sales/kpi-dashboard-spec.md` (optional)

### Phase 6: Role-play
- `.sales/role-play-transcripts/*.md` — count sessions. Print like: `role-play: 12 sessions across 4 personas`.

### Phase 7: Coach
- `.sales/recordings/<date>/*.{mp3,wav,m4a}` — count recordings without a matching `.transcript.txt` (these are pending) and count those already transcribed.
- `.sales/recordings-analysis/*.md` — count analysis files.
- `.sales/coach-notes.md` present?

Print like: `coach: 34 calls analyzed, 3 new recordings pending`.

### Sibling integration audit
- `.brand/voice.md` present in project root? If yes, print `brand sibling: detected, voice integration enabled`.
- `.marketing/leads/inbound/` present? If yes, print `marketing sibling: detected, inbound lead routing available`.
- `.greenfield/plan/data-architecture.md` present? If yes, print `greenfield sibling: detected, data-model alignment available`.

### Citation density audit (only if `--strict`)
For each `.sales/scripts/cold-call-v*.md` (latest version) and `.sales/scripts/objections-library.md`, use `Grep` to count inline `[Source` or `tier:` annotations. Spec Section 11 criterion 4 requires every script line to carry a citation. Flag any script with fewer than 1 citation per 5 lines as a strict failure.

### Framework citation audit on recordings (only if `--strict`)
For every file in `.sales/recordings-analysis/*.md`, use `Grep` to count distinct named-framework citations among: Gong, Blount, Voss, Cialdini, SPIN, Rackham, Challenger, Dixon, Adamson, Phil M. Jones, Josh Braun, Aaron Ross, Predictable Revenue, Keenan, Gap Selling, Sobczak, Mom Test, Fitzpatrick, Outreach, SalesLoft. Each analysis file must cite **at least 3 distinct frameworks**. List any file with fewer than 3 as a strict failure: `recordings-analysis/<file>: only N/3 framework citations`.

### Tier annotation audit (only if `--strict`)
`Grep` every file under `.sales/research/`, `.sales/scripts/`, `.sales/cadences/`, and `.sales/recordings-analysis/` for `tier: evidence`, `tier: scaffolding`, `tier: vocabulary-only`, `tier: educated-guess` annotations. Flag files that contain numerical claims (digits + %, ratios like "55:45") but no `tier:` annotations as strict failures.

### File header audit (only if `--strict`)
Spec Section 6 requires every markdown file in `.sales/` to carry a `Generated-by:`, `Depends-on:`, and `Sources-cited:` header block. Use `Grep` on each top-level `.sales/*.md` file and on every `.sales/research/*.md`, `.sales/scripts/*.md`, `.sales/cadences/*.md`, `.sales/recordings-analysis/*.md`, and `.sales/role-play-transcripts/*.md`. For each file, check that all three header keys are present in the first 30 lines. List any file missing any of the three keys as a strict failure.

## Output format

Print a one-screen table like this:

```
Sales Workflow Status
-----------------------------------------------
Phase 0: Init          [x]  context.md, recordings/README.md
Phase 1: Research      [x]  3/3 files
Phase 2: List          [x]  icp.md, strategy.md  (raw: 1 csv, processed: 1 csv)
Phase 3: Script        [x]  cold-call v3, cold-email-sequence, objections-library
Phase 4: Cadence       [x]  primary.md  (12 touches over 35 days)
Phase 5: CRM setup     [x]  HubSpot Free, manual-checklist, apply.py
Phase 6: Role-play     [x]  12 sessions across 4 personas
Phase 7: Coach         [x]  34 calls analyzed, 3 new recordings pending
-----------------------------------------------
Sibling integration:   brand=detected, marketing=absent, greenfield=detected
Next command:          /sales:coach --all   (3 new recordings since last run)
```

Always surface **exactly one** "Next command" line at the bottom. Choose the command that advances the earliest incomplete phase, OR if all phases are complete, point to the highest-leverage next action: typically `/sales:coach --all` if there are pending recordings, otherwise `/sales:role-play` for more practice reps.

If `--strict` is passed, also print: "Strict checks: script citation density=<pass|N failures>, recording framework citations=<N files passing / total>, tier annotations=<N failures>, header audit=<N failures>". Each strict failure must surface its filename and the specific gap so the user can fix it.

If `.sales/` does not exist, print: "No sales workflow found in this directory. Run `/sales:init` to start." and exit.

## Outputs

None — this skill is read-only and prints to stdout. Cited from spec Section 5 (`/sales:status` row) and Section 11 (success criteria used by `--strict`).

## Reentrancy

Fully reentrant, fully non-destructive, fully idempotent. Run at any time, any phase, any order. This skill never writes to `.sales/`, never creates `.sales/` if it does not exist, never modifies any file. If the user asks for state, this is the only skill that should run.
