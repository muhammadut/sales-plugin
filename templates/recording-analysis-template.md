# Recording Analysis — <date> — <recording-id>

Generated-by: call-coach (coach mode)
Depends-on: .sales/recordings/<date>/<file>.{mp3,wav,m4a}, .sales/recordings/<date>/<file>.transcript.txt, .sales/scripts/call-script-v<latest>.md, .sales/scripts/objection-library.md, ${CLAUDE_PLUGIN_ROOT}/knowledge/
Sources-cited: <list of knowledge/ files cited in coaching notes below>

## 1. Recording metadata

| Field | Value |
|---|---|
| Date | <YYYY-MM-DD> |
| Time | <HH:MM local> |
| Caller | <user name from icp.md or config> |
| Prospect | <name + business + city, redacted to first name only if recordings/ is committed> |
| Source | <Twilio webhook | Zoom export | phone recorder app | elocal_clone routing engine | manual upload> |
| Audio file | `<path relative to .sales/recordings/<date>/>` |
| Transcript file | `<same path with .transcript.txt suffix>` |
| Transcription provider | <Deepgram Nova-3 | AssemblyAI | Azure Speech | local Whisper> |
| Active script version | call-script-v<N>.md |
| Outcome | <sign-up | scheduled callback | soft no | hard no | wrong number | disconnect | voicemail> |
| Compliance flags | <list any: "did not identify self" | "did not mention recording" | "Quebec call without all-party consent" | "do-not-call list mention"> |

## 2. Transcript summary

<2-4 sentences. The arc of the call. Did the prospect pick up cold or after a voicemail? What was the dominant emotional tone — guarded, curious, hostile, friendly? Where did the call peak (best moment for the user) and where did it bottom (worst moment)? Did the caller follow the script or improvise?>

## 3. Quantitative metrics

| Metric | Value | Target / benchmark | Tier | Source |
|---|---|---|---|---|
| Total call duration | <mm:ss> | 5:50 for successful cold calls | evidence | Gong 100k cold calls |
| Talk-to-listen ratio (caller:prospect) | <X>:<Y> | 55:45 for cold calls | evidence | Gong talk-to-listen research |
| Average monologue duration | <ss>s | ~53s for successful cold calls | evidence | Gong 100k cold calls |
| Longest single monologue | <mm:ss> | <90s ceiling | evidence | Gong successful-call analysis |
| Words per minute (caller) | <integer> | 150-170 | scaffolding | Sobczak Smart Calling pacing guidance |
| Filler word density (per minute) | <integer> | <5/min | scaffolding | general practitioner guidance |
| Question count (caller) | <integer> | ≥6 for SPIN-shaped calls | evidence | Huthwaite SPIN 35k-call study |
| Script adherence (LLM diff %) | <integer>% | 60-80% (100% reads robotic) | scaffolding | Gong adherence-vs-outcome correlation |
| Outcome label | <see metadata> | sign-up rate target ≥10% per dial day | scaffolding | elocal_clone unit-economics target |

## 4. Opener evaluation

**Opener used (first 30 seconds, transcribed):**

> "<exact words>"

**Script's intended opener (from `.sales/scripts/call-script-v<N>.md` §3):**

> "<exact words>"

**Match:** <Yes — exact | Yes — paraphrased | No — drifted | No — abandoned>

**Coaching note:**

<2-3 sentences. If the opener matched and the prospect engaged, reinforce. If the opener matched and the prospect disengaged, suggest a variant from the script's alternates. If the caller drifted, name the drift and decide whether the drift was an improvement (worth promoting to the script in v<n+1>) or a regression (worth a coaching reminder).>

Citation: `[Gong cold call statistics — "How have you been?" has 6.6x higher success rate than "Did I catch you at a bad time?", tier: evidence]`

## 5. Objection patterns observed

For each objection raised by the prospect, record:

### Objection 1: "<exact words from prospect>"

| Field | Value |
|---|---|
| Timestamp | <mm:ss> |
| RBO category | <Reflexive | Bargaining | True Objection> |
| Was it in the objection library? | <Yes — `.sales/scripts/objection-library.md` §<N>` | No — NEW> |
| Caller's response | "<exact words>" |
| Response quality (1-5) | <integer> |
| Framework cited (or that should have been cited) | `[<framework + section>, tier: <tier>]` |

**Coaching note:** <one sentence on whether the response landed, what to do differently, and whether to add this objection to the library if it was new.>

### Objection 2: "<exact words from prospect>"

(repeat structure)

### Objection 3: "<exact words from prospect>"

(repeat structure)

## 6. Close evaluation

**Was a CTA attempted?** <Yes — soft | Yes — hard | No>

**CTA wording (if attempted):**

> "<exact words>"

**Prospect's response:**

> "<exact words>"

**Outcome:** <sign-up | callback scheduled | deferred | refused | ignored>

**Coaching note:**

<2-3 sentences. Was the CTA timed correctly (after the prospect signaled buying interest) or premature (mid-objection-handling)? Was the CTA shape right for the prospect's stated decision-making style? If no CTA was attempted, why — and was the omission correct (genuinely wrong moment) or a missed opportunity?>

Citation: `[Voss calibrated question — "What would have to be true for this to work for you?", tier: scaffolding]` OR `[Jeb Blount Fanatical Prospecting step 5 — ask for what you want, tier: scaffolding]`

## 7. Recommended script updates

The call-coach proposes script-version bumps when the same un-handled gap appears 3+ times across recordings. For this single recording, list any candidate updates with their occurrence count from `.sales/coach-notes.md`:

### Candidate update 1: <one-line description>

| Field | Value |
|---|---|
| Type | <new objection response | opener variant | value statement rewrite | close rewrite | talk-track addition> |
| Occurrences across `.sales/recordings-analysis/` to date | <integer> |
| Threshold to propose new script version | 3 |
| Status | <below threshold — log only | at threshold — propose v<n+1>` |
| Cited framework | `[<framework + section>, tier: <tier>]` |
| Suggested wording | "<exact proposed line>" |

### Candidate update 2: <one-line description>

(repeat structure)

## 8. Compliance and Canadian-context flags

For the elocal_clone first user, the call-coach surfaces specific signals:

| Flag | Triggered? | Note |
|---|---|---|
| Caller identified self by name + company | <Yes/No> | Required by professional norms |
| Call recording was disclosed | <Yes/No/N-A> | Canada is one-party consent federally; Quebec stricter — disclose if uncertain |
| Quebec call (all-party consent) | <Yes/No> | If Yes and recording was not disclosed, FLAG |
| French-language attempt by prospect | <Yes/No> | Triggers the bilingual Quebec persona suggestion for follow-up |
| Bill 96 mention | <Yes/No> | Trademark / French-language signage compliance signal |
| RBQ license question (Quebec plumbing) | <Yes/No> | Vertical-specific compliance check |
| Roto-Rooter / Mr. Rooter mention | <Yes/No> | Franchise rollup filter — note for ICP refinement |
| HomeStars / Angi / HomeAdvisor mention | <Yes/No> | Reverse-playbook signal — log specific complaint pattern |
| Do-not-call list mention | <Yes/No> | If Yes, immediately mark prospect as Lost and suppress from future cadences |

## 9. Aggregated coaching update

This recording's contribution to `.sales/coach-notes.md`:

- <one line: pattern observed in this call that reinforces or contradicts the running journal>
- <one line: specific coaching recommendation for the next dial session>

## 10. Next action

| Action | Owner |
|---|---|
| Mark prospect outcome in CRM | user |
| Add new objection to `.sales/scripts/objection-library.md` if applicable | user (after review) |
| Practice this scenario in `/sales:role-play` before next dial day | user (if quality score <3 on any dimension) |
| Apply proposed script update | user (after review of candidate updates above) |
| Re-run `/sales:coach` after next batch | user |
