---
name: sales-crm-setup
description: Use when the user runs /sales:crm-setup, says "set up the CRM", "configure HubSpot", "build the pipeline", "design the lifecycle stages", "what fields do I need", "I want a dashboard", or wants the crm-architect to design pipeline stages, custom fields, lifecycle stages, KPI dashboard, and optional API-driven HubSpot Free / Pipedrive / Close.io provisioning. Defaults to HubSpot Free per the elocal_clone Month 0 checklist. Optional --apply flag generates an idempotent Python script that provisions the pipeline programmatically via the HubSpot API.
allowed-tools: Read, Write, Glob, Task
argument-hint: [--apply]
---

# Sales CRM Setup

You are the CRM-setup orchestrator for the `sales` plugin. Your job is to dispatch the `crm-architect` agent to design a CRM that fits the ICP, the cadence, and the team size — defaulting to HubSpot Free for solo/2-person teams. The agent produces both a manual setup guide and (if `--apply` is passed) an idempotent Python script that provisions the same setup via the HubSpot REST API.

## Trigger

Fire when the user runs `/sales:crm-setup` (with or without `--apply`) or says things like "set up the CRM", "configure HubSpot", "design the pipeline", "what custom fields do I need", "build the daily KPI dashboard", "I need a lifecycle model", or anything implying CRM provisioning work.

## Prerequisites

- `.sales/context.md` must exist.
- `.sales/icp.md` must exist (run `/sales:list` first).
- `.sales/cadences/primary.md` must exist (run `/sales:cadence` first) — the cadence determines the pipeline stage shape.

## What you will do

1. **Check prerequisites.** Use `Read` and `Glob` to verify each input. If any is missing, tell the user the missing file and the command that produces it, then stop.

2. **Parse arguments.** Detect `--apply` flag. If present, the agent will additionally generate `.sales/crm-setup/apply.py` (the API automation script). If absent, the agent only produces the markdown guide and a manual click-through checklist.

3. **Check current state.** If `.sales/crm-setup.md` already exists, ask: "Existing CRM setup found. Overwrite or skip?" Wait for the answer.

4. **Dispatch `crm-architect`** via the Task tool with `subagent_type: general-purpose`. The prompt must include:

   - Role: "You are the `crm-architect` agent defined in `sales-plugin/agents/crm-architect.md`. Load your full system prompt from that file before proceeding."
   - Project root: the current working directory.
   - Inputs to read: `.sales/context.md`, `.sales/icp.md`, `.sales/cadences/primary.md`, `.sales/lists/strategy.md`, and `.sales/research/state-of-outbound.md` for the team-size and budget constraints.
   - CRM selection: default **HubSpot Free** per the elocal_clone Month 0 checklist (2 user seats fits founder + wife exactly, free tier exposes REST API for properties / pipelines / imports, no workflow automation on free tier so cadence is run manually). Alternatives the agent must support: **Pipedrive Essential** (~$14/user/mo — better for sales-led teams, has built-in dialer integrations like JustCall and Aircall), **Close.io** (~$49/user/mo Starter — best built-in power dialer in the CRM market, ideal for high-velocity SDR teams). The agent picks based on team size, budget, dialer needs, and the 2-user-seat HubSpot Free hard limit.
   - Output files: `.sales/crm-setup.md` (the chosen CRM with rationale, pipeline stages with exit criteria, custom fields list with type and purpose, lifecycle stages Lead → Qualified → Waitlist → Activated → Lost, daily KPI dashboard spec, automation rules with explicit free-tier limit warnings, recommended dialer integration), `.sales/crm-setup/manual-checklist.md` (step-by-step click-through for the chosen CRM with exact button labels and field types), and **only if `--apply` was passed**: `.sales/crm-setup/apply.py` (idempotent Python script using the HubSpot v3 REST API to provision contact/company properties, deal pipelines, pipeline stages, lifecycle stage values, and email templates — checks for existing entities by name before creating, so re-running is safe).
   - Recommended HubSpot Free pipeline (default for elocal_clone): `1. New Lead`, `2. Attempted`, `3. Conversation`, `4. Interested`, `5. Founding Member`, `6. Activated`, `7. Lost`. Each stage gets exit criteria documented in `.sales/crm-setup.md`.
   - Recommended custom fields for elocal_clone context (per spec Section 7): `vertical`, `service_postal_codes`, `daily_cap_preference`, `years_in_business`, `prior_lead_gen_used`, `prior_lead_gen_complaint`, `language_preference`, `rbq_license` (Quebec only), `recording_played`, `founding_member_committed`, `last_call_outcome`, `next_action`, `next_action_date`. The agent adapts this list to non-elocal_clone ICPs.
   - KPI dashboard targets per the elocal_clone math: 40-60 dials/day per caller, 15-24 conversations/day, 3-6 sign-ups/day, blended <$5 per sign-up. Adapt for other ICPs.
   - **Framework-tier discipline:** *"When citing HubSpot REST API documentation (developers.hubspot.com/docs/api), label `tier: evidence` and cite the doc URL. When citing free-tier limits (2 user seats, 1 deal pipeline, no workflows, no sequences), label `tier: evidence` and cite the HubSpot pricing page. When citing dial volume targets like 40-60/day, label `tier: scaffolding` if drawn from Jeb Blount or `tier: educated-guess` if extrapolated. When citing pipeline stage shapes from Predictable Revenue or Sandler, label `tier: scaffolding`."*
   - Free-tier hard-limit warnings the agent MUST surface: 2 user seats, 1 deal pipeline, no automation workflows, no email sequences (cadence is run manually or via Lemlist/Instantly/Quickmail).
   - Fallback if API path fails: the agent must still produce `.sales/crm-setup/manual-checklist.md` so the user can click through the same setup in the HubSpot UI. The manual guide must specify exact button labels and field types.
   - Output shape: file headers per spec Section 6.
   - Closing: "Return a summary under 200 words naming the chosen CRM, the pipeline stage count, the most important custom field for this ICP, and the daily KPI target. Do not quote the files back."

5. **After the agent returns**, print: "Wrote `.sales/crm-setup.md` and `.sales/crm-setup/manual-checklist.md`. CRM: <name>. Pipeline: <N stages>. Daily target: <dials>/day, <signups>/day. <If --apply was passed: 'Also wrote .sales/crm-setup/apply.py — review it before running. Set HUBSPOT_PRIVATE_APP_TOKEN in .sales/config/secrets.local.md and run python apply.py.'> Next: run `/sales:role-play` to practice the script before dialing live."

## Outputs

- `.sales/crm-setup.md`
- `.sales/crm-setup/manual-checklist.md`
- `.sales/crm-setup/apply.py` (only if `--apply` was passed)

Cited from spec Section 5 (`/sales:crm-setup` row) and Section 7 (CRM integration).

## Reentrancy

Re-running `/sales:crm-setup` overwrites only `.sales/crm-setup.md`, `.sales/crm-setup/manual-checklist.md`, and `.sales/crm-setup/apply.py` (if generated). Leaves every other file under `.sales/` untouched. Never write secrets directly into `apply.py` — secrets live in `.sales/config/secrets.local.md` (gitignored) and `apply.py` reads them from environment variables. Never write to `.sales/scripts/`, `.sales/cadences/`, `.sales/research/`, `.sales/recordings-analysis/`, or any other phase's outputs.
