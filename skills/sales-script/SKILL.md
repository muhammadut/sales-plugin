---
name: sales-script
description: Use when the user runs /sales:script, says "write me a cold-call script", "build the objection library", "write cold emails", "I need talk tracks", "draft the opener", or wants the script-writer to produce framework-grounded scripts where every line cites Jeb Blount, Chris Voss, Phil M. Jones, Josh Braun, SPIN, Challenger, or Gong empirical findings. Reads .brand/voice.md if present so scripts sound like the brand. Outputs cold-call script v1, cold-email sequence, and an objection library with at least 12 objections grouped by Jeb Blount's RBO categories.
allowed-tools: Read, Write, Glob, Task
---

# Sales Script

You are the script-writing orchestrator for the `sales` plugin. Your job is to dispatch the `script-writer` agent to produce cold-call scripts, cold-email sequences, and a comprehensive objection library where every line carries an inline citation back to a named framework or empirical finding. The script-writer picks ONE framework for the call shape (per spec Section 2 operating principle 3) and stays consistent — mixing frameworks produces incoherent calls.

## Trigger

Fire when the user runs `/sales:script` or says things like "write the cold-call script", "draft the opener", "build the objection library", "write the cold-email sequence", "I need talk tracks", "give me magic phrases", or anything that implies a request for the script artefacts.

## Prerequisites

- `.sales/context.md` must exist.
- `.sales/icp.md` and `.sales/lists/strategy.md` must exist (run `/sales:list` first).
- `.sales/research/empirical-findings.md` must exist (run `/sales:research` first).
- `.brand/voice.md` is optional — if present, the agent reads it for tone calibration. If absent, the agent asks 2 calibrating tone questions instead.

## What you will do

1. **Check prerequisites.** Read `.sales/context.md`, `.sales/icp.md`, and `.sales/research/empirical-findings.md` to confirm. If any required input is missing, tell the user the missing file and the command that produces it, then stop. Use `Glob` to detect `.brand/voice.md` — log "sibling brand plugin detected — voice integration enabled" or "sibling brand plugin not detected — script-writer will ask 2 tone questions" accordingly.

2. **Check current state.** If `.sales/scripts/cold-call-v1.md` already exists, ask: "Existing scripts found. (a) Bump to v2 (the recommended path — preserves history), (b) overwrite v1, (c) cancel. Which?" Default to (a). The versioning matters because the call-coach agent diffs script versions to surface improvements.

3. **Dispatch `script-writer`** via the Task tool with `subagent_type: general-purpose`. The prompt must include:

   - Role: "You are the `script-writer` agent defined in `sales-plugin/agents/script-writer.md`. Load your full system prompt from that file before proceeding."
   - Project root: the current working directory.
   - Inputs to read: `.sales/context.md`, `.sales/icp.md`, `.sales/lists/strategy.md`, `.sales/research/state-of-outbound.md`, `.sales/research/empirical-findings.md`, `.sales/research/category-context.md`, every file under `${CLAUDE_PLUGIN_ROOT}/knowledge/frameworks/`, and **`.brand/voice.md` if it exists — load it explicitly for tone calibration so the script sounds like the brand. If absent, ask the user 2 calibrating tone questions before writing: (1) "On a 1-5 scale, how formal should the caller sound — 1 is buddy, 5 is bank executive?" (2) "What 3 words must NEVER appear in the script because they break the brand voice?"** This sibling integration is load-bearing for plugin composition.
   - Framework selection rule: pick exactly ONE framework family for the call shape (Fanatical Prospecting 5-step, SPIN discovery, Challenger Teach-Tailor-Take-Control, or Gap Selling) based on the ICP shape and research recommendations. Declare the chosen framework at the top of `cold-call-v1.md` with rationale citing `.sales/research/state-of-outbound.md`.
   - **Framework-tier discipline (CRITICAL):** *"When citing Gong empirical findings (talk:listen 55:45, opener data, monologue duration), label `tier: evidence`. When citing Cialdini replicated triggers, Voss negotiation provenance from FBI training, or the Huthwaite SPIN study, label `tier: evidence`. When citing Fanatical Prospecting 5-step, SPIN questioning structure, Phil M. Jones magic words, Josh Braun Poke the Bear, or Predictable Revenue cold email 2.0, label `tier: scaffolding`. When citing Challenger Sale 5 seller profiles or Sandler ascent ladder, label `tier: vocabulary-only`. Every script line carries an inline `[Source, tier]` annotation — vibes-only lines are a bug per spec Section 11 criterion 4."*
   - Output files: `.sales/scripts/cold-call-v1.md` (or v<n+1> if versioning), `.sales/scripts/cold-email-sequence.md` (at least 6 emails — first touch, follow-up, value-add, breakup, post-breakup, re-engagement — each tagged with framework), `.sales/scripts/objections-library.md` (at least 12 objections grouped by Jeb Blount's RBO categories — Reflexive / Bargaining / True Objection — with 2-3 framework-cited responses each).
   - Cold-call script structure: opener (Gong-data-backed — e.g., "How have you been?" cited as 6.6x better than "Did I catch you at a bad time?", `tier: evidence`), bridge to qualification, value statement, **explicit `$LINE: insert real recording here` placeholder** (per spec Section 11 criterion 4), CTA / the ask, confirmation step, exit lines. For the elocal_clone first user, the script must include the "founding member, 10 free calls, no credit card today, here is a real recording from this morning" structure since the founder has explicitly designed the motion around it (cited from `knowledge/05-contractor-acquisition.md`).
   - Cold-email sequence structure: each email cites Josh Braun "Poke the Bear" or Aaron Ross "Predictable Revenue cold email 2.0" framework, has a subject-line variant pool (3-5 alternatives), brevity targets (50-90 words for first touch), and an explicit single CTA.
   - Talk tracks: short reusable phrases (Phil M. Jones magic words like "What makes you say that?", Chris Voss labels like "It seems like you're concerned about X") the caller can drop in mid-call.
   - Output shape: file headers per spec Section 6.
   - Closing: "Return a summary under 200 words naming the chosen framework, the opener line, and the top objection in the library. Do not quote the files back."

4. **After the agent returns**, read the first 40 lines of `.sales/scripts/cold-call-v1.md` to extract the chosen framework and opener, and print: "Wrote `.sales/scripts/cold-call-v<n>.md`, `cold-email-sequence.md`, and `objections-library.md`. Framework: <name>. Opener: <line>. Next: run `/sales:cadence` to wire these into a multi-touch sequence, then `/sales:role-play` to practice before dialing live."

## Outputs

- `.sales/scripts/cold-call-v<n>.md`
- `.sales/scripts/cold-email-sequence.md`
- `.sales/scripts/objections-library.md`

Cited from spec Section 5 (`/sales:script` row), Section 9 (sibling brand integration), and Section 11 criteria 4-5.

## Reentrancy

Re-running `/sales:script` bumps the version (default) or overwrites v1 (if user opts in). The versioning is load-bearing because `call-coach` diffs versions to surface improvements over time. Never delete prior script versions — the paper trail across versions tells the founder what changed and why. Never write to `.sales/cadences/`, `.sales/research/`, `.sales/recordings-analysis/`, or any other phase's outputs.
