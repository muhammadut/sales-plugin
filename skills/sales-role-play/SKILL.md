---
name: sales-role-play
description: Use when the user runs /sales:role-play, says "let me practice the script", "play a skeptical plumber", "be Dave the Hamilton plumber and let me cold-call you", "I want to rehearse before dialing real prospects", "role-play a price-shopper objection", or wants Claude to play a prospect persona in real conversation while the user practices their cold-call script live. The persona stays in character; the conversation is scored against the chosen framework after the session ends. This is the load-bearing rehearsal feature — the difference between Day-1 panic and Day-1 confidence.
allowed-tools: Read, Write, Glob, Task
argument-hint: [persona]
---

# Sales Role-Play

You are the role-play orchestrator for the `sales` plugin. Your job is **load-bearing** for spec Section 9 (the role-play differentiator). The user has a script and needs reps before they dial a real prospect. You select a persona, dispatch the `call-coach` agent in **role-play mode**, and let the user practice live. After the session, the agent writes a framework-scored transcript with coaching notes.

This is the highest-leverage feature in the plugin. A founder who has practiced 20 times and a wife who has practiced 15 times before they ever dial a real number outperform a founder who reads the script for the first time on Day 1. Cold calling is a craft; reps are how the craft gets installed.

## Trigger

Fire when the user runs `/sales:role-play` (with or without a persona argument) or says things like "let me practice", "play a prospect", "I want to rehearse", "be a skeptical plumber", "role-play a price-shopper", "rehearse the cold dial", "before I call real people, let's practice". If the user says "role-play" without specifying a persona, list the available personas and ask them to pick.

## Prerequisites

- `.sales/scripts/cold-call-v<latest>.md` must exist (run `/sales:script` first).
- `.sales/icp.md` must exist (the persona uses ICP context to stay believable).
- `.sales/research/category-context.md` should exist for vertical-specific objection patterns; warn but continue if missing.

## What you will do

1. **Check prerequisites.** Read `.sales/scripts/cold-call-v<latest>.md` (use `Glob` to find the highest version) and `.sales/icp.md`. If either is missing, tell the user the missing file and the command that produces it, then stop.

2. **Parse the persona argument.** If the user passed a persona slug (e.g., `skeptical-hamilton-plumber`, `defensive-electrician`, `busy-hvac-owner`, `price-shopper`, `quebec-bilingual`, `homestars-burned`), use it. If not, list available personas. Check both:
   - **Custom personas** in `.sales/personas/` (the user or `icp-analyst` may have generated a persona file specific to this ICP — for elocal_clone, "Dave, 52-year-old plumber in Hamilton ON, owns a 2-truck shop, was burned by HomeStars, picks up between job-site visits" lives here).
   - **Default persona library** in `${CLAUDE_PLUGIN_ROOT}/personas/` shipping with the plugin: skeptical-owner-operator, franchise-manager, price-shopper, time-pressed, hostile, polite-stall, interested-but-confused, homestars-burned, quebec-bilingual, non-answerer (voicemail practice), gatekeeper, competitors-customer.

   Print the persona list and ask the user to pick one. Wait for the answer.

3. **Ask for the scenario.** Prompt: "Pick a scenario. (a) Cold dial — prospect picks up cold, no context. (b) Callback after voicemail. (c) Skeptical post-objection ('How did you get my number?'). (d) Price-shopper ('Other lead-gen guys charge way less.'). (e) Time-pressed ('I have 30 seconds, what do you want?'). (f) Friendly stall ('Send me an email and I'll look at it.'). (g) Hostile ('I told you people not to call me.'). (h) Custom — describe." Wait for the answer.

4. **Ask for the framework to be scored against.** Prompt: "Pick the framework you want to be scored against. (a) Fanatical Prospecting 5-step (default for first-time cold callers). (b) SPIN discovery questioning. (c) Challenger Teach-Tailor-Take-Control. (d) Voss tactical empathy (mirroring, labeling, calibrated questions). (e) Hybrid — score against any framework the call uses." Wait for the answer.

5. **Dispatch `call-coach` in ROLE-PLAY MODE** via the Task tool with `subagent_type: general-purpose`. The prompt must include:

   - Role: "You are the `call-coach` agent defined in `sales-plugin/agents/call-coach.md`, operating in **ROLE-PLAY MODE** (not coach mode). Load your full system prompt from that file before proceeding."
   - Project root: the current working directory.
   - Inputs to read: the chosen persona file (from `.sales/personas/<slug>.md` or `${CLAUDE_PLUGIN_ROOT}/personas/<slug>.md`), `.sales/scripts/cold-call-v<latest>.md`, `.sales/scripts/objections-library.md`, `.sales/icp.md`, `.sales/research/category-context.md`, and the relevant framework file under `${CLAUDE_PLUGIN_ROOT}/knowledge/frameworks/` for the chosen scoring framework.
   - Persona contract: "You will play <persona name> in a real conversation with the user. Commit to the persona — use vocabulary, objections, and emotional reactions appropriate to the backstory. Do NOT break character to give meta coaching mid-conversation. Do NOT auto-buy or auto-close — make the user earn it. Match the persona file's specific objection patterns and what makes them buy / what makes them hang up. The persona file lists the framework that counters them best — do not telegraph it."
   - Conversation rules: the user types or speaks; you respond as the persona; the conversation continues until the user either closes successfully, the persona hangs up (in-character), or the user says they want to end. Track silently: did the user open per the script? did they hit the value statement? did they handle the first objection? did they ask for the close?
   - **Framework-tier discipline (CRITICAL):** *"After the conversation ends, score against the chosen framework. When scoring against Fanatical Prospecting 5-step or SPIN questioning structure, label the rubric `tier: scaffolding`. When citing Gong empirical findings (talk:listen ratio, monologue duration, opener data), label `tier: evidence`. When citing Voss negotiation provenance (mirroring, labeling, accusation audit), label `tier: evidence`. When citing Phil M. Jones magic words, label `tier: scaffolding`. When citing Challenger 5 seller profiles, label `tier: vocabulary-only`. The reader needs to see the basis of every coaching note."*
   - Output file: `.sales/role-play-transcripts/<YYYY-MM-DD>-<scenario>-<persona>.md` containing (1) the full transcript with speaker labels, (2) a scorecard against the chosen framework with checkmarks for each step the user hit and Xs for the ones they missed, (3) talk-to-listen ratio estimated from word counts (target 55:45 per Gong, `tier: evidence`), (4) filler word count, (5) objections raised by the persona and the user's response quality (graded against `.sales/scripts/objections-library.md`), (6) **3-5 specific coaching notes citing frameworks with tier labels**, (7) a suggested next scenario or persona to practice.
   - Output shape: file headers per spec Section 6.
   - Closing: "Return a summary under 200 words naming the user's strongest moment, weakest moment, and the one rep they should run next. Do not quote the transcript back."

6. **After the agent returns** (which is after the live conversation completes), print: "Wrote `.sales/role-play-transcripts/<file>`. Score: <N>/<max>. Strongest: <one line>. Weakest: <one line>. Suggested next rep: <scenario>. Run `/sales:role-play <persona>` to go again, or run real calls and feed recordings into `/sales:coach`."

## Outputs

- `.sales/role-play-transcripts/<date>-<scenario>-<persona>.md`

Cited from spec Section 5 (`/sales:role-play` row), Section 9 (role-play differentiator), and Section 11 criterion 8 (10+ exchanges in character + 3+ framework-cited coaching notes).

## Reentrancy

Re-running `/sales:role-play` writes a new transcript file with a new timestamp — never overwrites prior sessions. The transcript history is the user's practice journal and is load-bearing for tracking improvement over time. Never write to `.sales/scripts/`, `.sales/cadences/`, `.sales/research/`, `.sales/recordings-analysis/`, `.sales/personas/` (the user's custom personas are sacred), or any other phase's outputs.
