---
name: sales-list
description: Use when the user runs /sales:list, says "build my prospect list strategy", "how do I source contractors", "find me 500 plumbers in Ontario", "what scraping tool should I use", or wants the icp-analyst to produce ICP firmographics, sourcing recipes (Outscraper, Apollo, Clay, ZoomInfo, manual), dedup rules, lead scoring, and disqualification heuristics. Output is a list-strategy doc plus actionable scrape recipes — not the actual list. The user runs the recipes; the plugin only produces the playbook.
allowed-tools: Read, Write, Glob, Task
---

# Sales List

You are the list-strategy orchestrator for the `sales` plugin. Your job is to dispatch the `icp-analyst` agent to produce a defensible Ideal Customer Profile and the actionable sourcing recipes that turn it into a real list. The agent does not scrape — it writes the playbook the user (or a `--scrape` flag in a future version) will run.

## Trigger

Fire when the user runs `/sales:list` or says things like "build my list strategy", "how should I source prospects", "what scraping tools fit my ICP", "give me dedup rules", "score my list", "find me 500 contractors", or anything that implies a request for the list-sourcing playbook.

## Prerequisites

- `.sales/context.md` must exist (run `/sales:init` first if missing).
- `.sales/research/state-of-outbound.md` and `.sales/research/category-context.md` should exist (run `/sales:research` first). If missing, warn the user and offer to proceed without research grounding — but recommend they run `/sales:research` first for better citations.

## What you will do

1. **Check prerequisites.** Read `.sales/context.md` to confirm. Use `Glob` to check `.sales/research/*.md`. If research is missing, ask: "No research found. Run `/sales:research` first (recommended), or proceed without it (lower-quality citations)?" Wait for the answer.

2. **Check current state.** If `.sales/icp.md` or `.sales/lists/strategy.md` already exists, ask: "Existing list strategy found. Overwrite or skip?" Wait for the answer.

3. **Dispatch `icp-analyst`** via the Task tool with `subagent_type: general-purpose`. The prompt must include:

   - Role: "You are the `icp-analyst` agent defined in `sales-plugin/agents/icp-analyst.md`. Load your full system prompt from that file before proceeding."
   - Project root: the current working directory.
   - Inputs to read: `.sales/context.md`, `.sales/research/state-of-outbound.md`, `.sales/research/category-context.md`, `.sales/research/empirical-findings.md`, every file under `${CLAUDE_PLUGIN_ROOT}/knowledge/frameworks/` (especially `predictable-revenue.md` and `mom-test.md` for ICP discipline), and `.greenfield/plan/data-architecture.md` if it exists (sibling integration — degrade gracefully). For elocal_clone, also reference `knowledge/05-contractor-acquisition.md` if pre-seeded into context.
   - **Framework-tier discipline:** *"When citing Predictable Revenue's ICP definition or Mom Test interview rules, label `tier: scaffolding`. When citing Outscraper/Apollo/Clay throughput data from vendor documentation, label `tier: evidence` and cite the vendor doc URL. When citing Aaron Ross's 150-250 emails/week target, label `tier: scaffolding`. The reader needs to see the basis of every recommendation."*
   - Output files: `.sales/icp.md` (locked ICP definition with firmographics, inclusion/exclusion criteria, persona shape, named buyer), `.sales/lists/strategy.md` (source ranking with cost and expected throughput per source, dedup rules, lead score formula, disqualification heuristics — especially the "remove HomeAdvisor patterns" reverse-playbook items from `.sales/research/category-context.md`).
   - Sourcing recipes the agent must include with cost and throughput estimates: **Outscraper** (~$40/mo for ~5,000 Google Maps records — best for blue-collar / local services contractors; cite the elocal_clone Hamilton plumbing example if context matches), **Apollo.io** (~$49/mo Basic, ~$79/mo Pro — best for B2B SaaS / mid-market with verified emails), **Clay** (~$149/mo starter — best for enrichment chains and hyper-personalization), **ZoomInfo** (enterprise pricing $15k+/yr — only for enterprise sales), **LinkedIn Sales Navigator** (~$99/mo — best for executive prospecting), **manual sources** (supply-house posters, Facebook groups, trade-association directories, Yellow Pages CSV exports — free, slow, high-intent). Each recipe must list the exact query patterns, expected record counts per query, post-processing steps (CSV cleanup, dedup against an existing CRM, fuzzy-match on phone numbers), and enrichment additions (NeverBounce / ZeroBounce for email verification, Phantombuster for LinkedIn enrichment).
   - Output shape: file headers per spec Section 6 (`Generated-by:`, `Depends-on:`, `Sources-cited:`).
   - Closing: "Return a summary under 200 words naming the top 2 sources for this ICP, expected total list size achievable in week 1, and the disqualification rule the user must enforce hardest. Do not quote the files back."

4. **After the agent returns**, read the first 40 lines of `.sales/lists/strategy.md` and print: "Wrote `.sales/icp.md` and `.sales/lists/strategy.md`. Top sources: <2 names>. Estimated list size week 1: <number>. Next: run `/sales:script` to dispatch `script-writer` with this ICP as input. The user is responsible for actually running the scrape — the strategy doc tells you exactly how."

## Outputs

- `.sales/icp.md`
- `.sales/lists/strategy.md`

Cited from spec Section 5 (`/sales:list` row).

## Reentrancy

Re-running `/sales:list` overwrites only `.sales/icp.md` and `.sales/lists/strategy.md` and leaves every other file under `.sales/` untouched. Never delete files in `.sales/lists/raw/` — those are the user's actual scraped data and are sacred (and PII-sensitive). Never delete `.sales/lists/processed/` files. Never write to `.sales/research/`, `.sales/scripts/`, `.sales/cadences/`, or any other phase's outputs.
