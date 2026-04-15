# Sales Workflow

## Phase diagram

```
      ┌──────────────┐
      │  empty dir   │
      └──────┬───────┘
             │
             │ /sales:init
             ▼
      ┌──────────────┐
      │   INIT       │──── interview (one question at a time)
      │              │──── .sales/icp.md
      │              │──── .sales/recordings/README.md
      │              │──── .sales/config/config.yaml
      └──────┬───────┘
             │
             │ /sales:research
             ▼
      ┌──────────────┐
      │  RESEARCH    │──── sales-researcher
      │              │──── .sales/research.md
      └──────┬───────┘
             │
             │ /sales:list
             ▼
      ┌──────────────┐
      │    LIST      │──── icp-analyst
      │              │──── .sales/list-strategy.md
      │              │──── .sales/lists/sourcing.md
      └──────┬───────┘
             │
             │ /sales:script
             ▼
      ┌──────────────┐
      │   SCRIPT     │──── script-writer
      │              │──── .sales/scripts/call-script-v1.md
      │              │──── .sales/scripts/objection-library.md
      │              │──── .sales/scripts/talk-tracks.md
      └──────┬───────┘
             │
             │ /sales:cadence
             ▼
      ┌──────────────┐
      │   CADENCE    │──── cadence-designer
      │              │──── .sales/cadences/primary.md
      │              │──── .sales/cadences/email-templates.md
      └──────┬───────┘
             │
             │ /sales:crm-setup [--apply]
             ▼
      ┌──────────────┐
      │  CRM SETUP   │──── crm-architect
      │              │──── .sales/crm-setup.md
      │              │──── .sales/kpi-dashboard-spec.md
      │              │──── (--apply) .sales/crm-setup/apply.py
      └──────┬───────┘
             │
             │ /sales:role-play  (loop)
             ▼
      ┌──────────────┐
      │  ROLE-PLAY   │──── call-coach (role-play mode)
      │              │──── .sales/role-play-transcripts/<date>-<scenario>.md
      └──────┬───────┘
             │
             │ user makes real cold calls, drops recordings
             ▼
      ┌──────────────────────────────────────┐
      │  USER RECORDS REAL CALLS              │
      │  (Twilio / Zoom / phone recorder /    │
      │   elocal_clone routing engine)        │
      │  drops files into                     │
      │  .sales/recordings/<date>/            │
      └──────┬───────────────────────────────┘
             │
             │ /sales:coach
             ▼
      ┌──────────────┐
      │    COACH     │──── transcribe (Deepgram | Azure Speech | local Whisper)
      │              │──── call-coach (coach mode)
      │              │──── .sales/recordings-analysis/<date>.md
      │              │──── .sales/coach-notes.md
      └──────┬───────┘
             │
             ├── ACCEPT script update → .sales/scripts/call-script-v<n+1>.md
             │            → loop back to ROLE-PLAY (practice the new lines)
             │            → resume real calling with v<n+1>
             │
             └── REJECT update → keep current script, more reps before re-evaluating

      ┌──────────────┐
      │   STATUS     │──── /sales:status (reentrant, any time)
      └──────────────┘
```

`/sales:status` is reentrant and can be run at any point. It prints which phases are complete, which files exist, and which blocking inputs are missing. It dispatches no agents — it walks the `.sales/` tree.

## Time budget

| Phase | Command | Actor | Wall time (plugin-side) | Wall time (user-side) |
|---|---|---|---|---|
| Init | `/sales:init` | user interview | 0 | 8-15 min |
| Research | `/sales:research` | `sales-researcher` | 3-6 min | 0 |
| List | `/sales:list` | `icp-analyst` | 3-6 min | 0 |
| List scrape (optional) | (user runs Outscraper / Apollo) | user | 0 | 15-45 min |
| Script | `/sales:script` | `script-writer` | 4-8 min | 0 |
| Cadence | `/sales:cadence` | `cadence-designer` | 3-6 min | 0 |
| CRM setup | `/sales:crm-setup` | `crm-architect` | 4-8 min | 0 |
| CRM apply | `/sales:crm-setup --apply` | `crm-architect` (Bash) | 10-15 min | 0 |
| CRM manual setup | (user follows checklist) | user | 0 | 20-45 min |
| Role-play | `/sales:role-play` | `call-coach` (role-play mode) | per session 15-45 min | per session 15-45 min |
| Real calling | (user dials) | user | 0 | 2-4 hours/day |
| Coach | `/sales:coach` | `call-coach` (coach mode) | 5-15 min per recording batch | 0 |
| Status | `/sales:status` | (skill walks tree) | <1 min | 0 |
| **Plugin-side total (init through CRM)** | | | **~30-60 min** | |
| **User-side total to first dial** | | | | **~3-5 hours including 2 role-play reps** |

Plugin-side time is deterministic and bounded. User-side time is dominated by role-play practice (the secret weapon — every script gets reps before touching a real prospect) and real call volume. The plugin imposes no schedule; it imposes only a discipline. See spec §11 for the elocal_clone-specific user-journey table.

## State directory tree

All state lives in `.sales/` inside the **target project**, never inside the plugin.

```
<target-project>/
└── .sales/
    ├── icp.md                               # /sales:init output — canonical headers, parsed by downstream agents
    ├── research.md                          # /sales:research output (sales-researcher)
    ├── list-strategy.md                     # /sales:list output (icp-analyst)
    ├── lists/
    │   ├── sourcing.md                      # icp-analyst's scraping recipes
    │   ├── raw/                             # CSV dumps from Outscraper/Apollo (gitignored — PII)
    │   │   └── hamilton-plumbing-2026-04-15.csv
    │   └── processed/                       # deduped, scored, ready for CRM import
    │       └── hamilton-plumbing-2026-04-15.csv
    ├── scripts/
    │   ├── call-script-v1.md                # versioned, never overwritten
    │   ├── call-script-v2.md                # call-coach proposes new versions; user reviews diffs
    │   ├── call-script-v3.md
    │   ├── objection-library.md             # 12+ objections by Blount RBO categories
    │   └── talk-tracks.md                   # Phil M. Jones magic words, Voss labels, drop-in phrases
    ├── cadences/
    │   ├── primary.md                       # 8-16 touches over 21+ days
    │   └── email-templates.md               # 6+ emails, tagged by framework
    ├── crm-setup.md                         # chosen CRM, pipeline, fields, lifecycle stages
    ├── crm-setup/
    │   ├── apply.py                         # optional API-driven HubSpot bootstrap
    │   └── manual-checklist.md              # fallback if --apply not used
    ├── kpi-dashboard-spec.md                # daily dashboard the founder + wife look at every morning
    ├── personas/                            # project-custom personas generated by icp-analyst
    │   └── dave-the-hamilton-plumber.md
    ├── role-play-transcripts/               # /sales:role-play output (call-coach role-play mode)
    │   ├── 2026-04-15-cold-dial.md
    │   ├── 2026-04-16-skeptical-quebec.md
    │   └── ...
    ├── recordings/                          # USER drops audio here from Twilio/Zoom/phone recorder
    │   ├── README.md                        # written by /sales:init
    │   ├── 2026-04-20/
    │   │   ├── call-001.mp3                 # gitignored
    │   │   ├── call-001.transcript.txt      # generated by /sales:coach
    │   │   ├── call-002.mp3
    │   │   └── call-002.transcript.txt
    │   └── 2026-04-21/
    │       └── ...
    ├── recordings-analysis/                 # /sales:coach output (call-coach coach mode)
    │   ├── 2026-04-20.md                    # one file per day batch
    │   └── 2026-04-21.md
    ├── coach-notes.md                       # running coaching journal — patterns aggregated across sessions
    ├── closed-lost-reasons.md               # written for marketing plugin to consume (attribution)
    └── config/
        ├── config.yaml                      # transcription provider, dialer, CRM choice
        ├── secrets.local.md                 # API keys (gitignored, YAML frontmatter)
        └── README.md                        # explains config schema
```

`/sales:init` writes a `.gitignore` snippet that gitignores `lists/raw/`, `recordings/**/*.{mp3,wav,m4a}`, and `secrets.local.md` while keeping every markdown artefact committable. Every markdown file has a fixed header: `Generated-by:`, `Depends-on:`, `Sources-cited:`. The `sales-status` skill validates these on every run.

## The recording loop

The closed-loop differentiator. The plugin replicates the core Gong / Chorus / ExecVision motion for the cost of a Deepgram subscription.

```
user records cold call (Twilio webhook | Zoom export | phone recorder app | elocal_clone routing engine)
        │
        ▼
user drops audio file into .sales/recordings/<date>/
   OR passes path to /sales:coach <file>
        │
        ▼
/sales:coach
        │
        ▼
read .sales/config/config.yaml → pick transcription provider
   default: Deepgram (fast, $0.25/hr)
   when greenfield Azure stack detected: Azure Speech ($1.00/hr, all-Azure)
   privacy mode: local Whisper ($0, slow)
        │
        ▼
transcribe → write <file>.transcript.txt next to audio
        │
        ▼
dispatch call-coach (coach mode) with:
   - transcript
   - active script (.sales/scripts/call-script-v<latest>.md)
   - objection library
   - knowledge bundle (frameworks + reports)
        │
        ▼
Gong-style analysis:
   - talk:listen ratio (target 55:45 for cold calls, tier: evidence)
   - monologue duration (target ~53s, tier: evidence)
   - opener detection (vs script's opener)
   - objection classification (RBO categories, tier: scaffolding)
   - filler word density
   - outcome label
        │
        ▼
write .sales/recordings-analysis/<date>.md
   per-call breakdown with framework-cited coaching notes
        │
        ▼
aggregate into .sales/coach-notes.md
   running patterns across all sessions
        │
        ▼
when same un-handled objection appears 3+ times:
   propose .sales/scripts/call-script-v<n+1>.md
   surface diff against current version
        │
        ▼
user reviews → ACCEPT (commit v<n+1>) | SHOW DIFF | REJECT (keep current)
        │
        ▼
loop back to ROLE-PLAY to practice the new lines, then resume real calling
```

This is the loop that turns a solo team into something like a Gong-instrumented sales floor for near-zero cost. See `docs/design.md` for the full architectural defense.

## The role-play loop

The other differentiator. Cold calling is a craft that decays without practice; role-play is the cheapest insurance against burning the first 50 dials on a script the founder cannot execute.

```
user runs /sales:role-play
        │
        ▼
skill prompts: "Pick a scenario."
   - cold dial
   - callback after voicemail
   - skeptical post-objection ("How did you get my number?")
   - price-shopper
   - time-pressed
   - friendly stall
   - hostile
   - custom (user describes)
        │
        ▼
skill prompts: "Pick the framework you want to be scored against."
   - Fanatical Prospecting 5-step (tier: scaffolding)
   - SPIN discovery (tier: evidence — 35k-call Huthwaite study)
   - Challenger Teach-Tailor-Take-Control (tier: vocabulary-only)
   - Voss tactical empathy (tier: scaffolding)
   - Hybrid (allow any)
        │
        ▼
skill loads persona file (preference order):
   1. .sales/personas/<custom>.md (project-specific, generated by icp-analyst)
   2. ${CLAUDE_PLUGIN_ROOT}/personas/<archetype>.md (bundled library)
        │
        ▼
dispatch call-coach (role-play mode)
   commit to persona — vocabulary, objections, decision-making style
   refuse to break character
        │
        ▼
user practices live (text mode default; voice mode is a future flag)
   Claude tracks: opener used? value statement? objection handled? close?
        │
        ▼
conversation ends (sign-up | hang-up | user gives up)
        │
        ▼
write .sales/role-play-transcripts/<date>-<scenario>.md
   - full transcript
   - scorecard against chosen framework (✓/✗ per step + 1-5 quality)
   - talk:listen ratio (estimated from word counts)
   - filler word count
   - 3-5 framework-cited coaching notes
   - suggested next scenario
        │
        ▼
user iterates the script before real calls
```

Every script gets reps before it touches a real prospect.

## Reentrancy contract

Every skill is idempotent on its inputs. Re-running a skill overwrites only its own outputs and leaves unrelated files untouched. Concretely:

- `/sales:init` writes `.sales/icp.md`, the directory tree, the `.gitignore` snippet, and `recordings/README.md`. If `.sales/icp.md` exists, the skill prompts resume / edit / restart and never silently overwrites.
- `/sales:research` overwrites `.sales/research.md` and nothing else.
- `/sales:list` overwrites `.sales/list-strategy.md` and `.sales/lists/sourcing.md` and nothing else.
- `/sales:script` writes `.sales/scripts/call-script-v<next>.md`, `.sales/scripts/objection-library.md`, and `.sales/scripts/talk-tracks.md`. NEVER overwrites a previous version of the call script — always bumps the version. The user gets a clean version history.
- `/sales:cadence` overwrites `.sales/cadences/primary.md` and `.sales/cadences/email-templates.md`.
- `/sales:crm-setup` overwrites `.sales/crm-setup.md`, `.sales/kpi-dashboard-spec.md`, and (when `--apply`) `.sales/crm-setup/apply.py`. The `--apply` run is itself idempotent against the HubSpot API — the script checks for existing properties and pipeline stages by name before creating.
- `/sales:role-play` writes a new file in `.sales/role-play-transcripts/`, never overwrites previous transcripts.
- `/sales:coach` writes `.sales/recordings-analysis/<date>.md` (overwrites any prior run for that date), updates `.sales/coach-notes.md` (append-only — never deletes prior entries), and may propose a new `.sales/scripts/call-script-v<n+1>.md` (always bumps version, never overwrites).
- `/sales:status` writes nothing. Read-only.

Verify by running `git diff` after a re-run. Any unexpected file touch is a bug.

The user can exit and resume at any phase. `/sales:status` shows where they are.

## Three unhappy paths

**Unhappy path 1 — script dies on first contact.** User runs `/sales:init` through `/sales:cadence`, role-plays the script twice, dials the first 20 prospects, gets rejected by all 20 (no sign-ups, no soft maybes, no callbacks). The first instinct is to refine the script. The right move is to backtrack to the ICP. User re-runs `/sales:init` (resume mode) to revise `.sales/icp.md` — wrong vertical, wrong persona within the vertical, wrong geography. Then re-runs `/sales:research` and `/sales:list` against the new ICP. The old `.sales/scripts/call-script-v1.md` is preserved (versioned), so the audit trail is intact. Plugin tolerates this — the reentrancy contract guarantees nothing is lost. Total backtrack cost: ~30 minutes.

**Unhappy path 2 — founder is talking 80%.** First batch of 30 real recordings drops into `.sales/recordings/2026-04-20/`. User runs `/sales:coach`. Analysis reveals the founder's average talk-to-listen ratio is 80:20 (Gong target is 55:45 for cold calls, `tier: evidence`). The founder is monologuing the value statement and never asking discovery questions. The fix is not "write a new script" — the script is fine. The fix is more reps before the next dial day. User runs `/sales:role-play` 5 more times with the price-shopper, time-pressed, and skeptical scenarios, focusing on listening more and bridging into SPIN questions. The next dial day's recordings (run through `/sales:coach` again) show the ratio drift toward 60:40. Plugin supports this loop: role-play → coach → role-play → coach until the metric closes the gap.

**Unhappy path 3 — HubSpot Free hits its limits.** Six weeks in, the founder has 2,500 contacts in HubSpot Free, both seats are used (founder + wife), and they need workflow automation to follow up consistently. Free tier doesn't have workflows. User runs `/sales:crm-setup` again and the crm-architect picks Pipedrive Essential ($14.90/user/month) instead, generates a Pipedrive-flavored `.sales/crm-setup.md`, and writes a migration checklist that maps HubSpot fields to Pipedrive fields. The Pipedrive `apply.py` provisions the same pipeline shape against Pipedrive's API. The old HubSpot setup files are preserved as `.sales/crm-setup-hubspot-archived.md`. Plugin tolerates the CRM swap — `crm-architect` is the only agent that needs to know about it, and the rest of `.sales/` is CRM-agnostic.

All three paths are supported. The plugin tolerates the user's real workflow.
