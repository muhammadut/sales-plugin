# Cadence — <cadence-name> — v<N>

Generated-by: cadence-designer
Depends-on: .sales/icp.md, .sales/scripts/call-script-v<latest>.md, .sales/scripts/objection-library.md, .sales/research.md
Sources-cited: <comma-separated list of knowledge/reports/ files cited below>

## Cadence summary

- **Total touches:** <N> (target range 8-16 per Outreach.io and SalesLoft Big Book of Sales Cadences research, `tier: evidence`)
- **Total span:** <N> days (target ≥21 days — 80% of sales require 5+ touches, IRC Sales Solutions, `tier: scaffolding`)
- **Channel mix:** call / email / LinkedIn / voicemail / text — at least 3 distinct channels
- **Persona target:** <one sentence pointing at .sales/icp.md>
- **Exit criteria:** prospect signs up | prospect explicitly opts out | full cadence completed without engagement → move to nurture
- **Re-entry rule:** prospects who complete the cadence without engagement re-enter after <N> days (default 90 days, cite Jeb Blount 30-Day Rule + nurture conventions)

## The 12-touch default sequence

This is the default for B2B owner-operator outbound (the elocal_clone first-user shape). Adjust touch count, channel mix, and timing per the icp-analyst's persona analysis.

### Touch 1 — Day 1 — Call + email

| Field | Value |
|---|---|
| Day | 1 |
| Channel | Phone (primary) + Email (same day backup) |
| Goal | First conversation OR voicemail + email pickup |
| Content reference | `.sales/scripts/call-script-v<latest>.md` opener; `.sales/cadences/email-templates.md` Touch-1 email |
| Expected connect rate | 15-25% on first dial (Gong cold call statistics, `tier: evidence`) |
| Citation | `[Outreach.io cadence research — Day 1 dual-channel touch lifts engagement 1.6x over single-channel, tier: evidence]` |
| Exit if | Prospect picks up and verbally opts out → mark Lost; otherwise advance |

### Touch 2 — Day 3 — Email

| Field | Value |
|---|---|
| Day | 3 |
| Channel | Email |
| Goal | Re-anchor with a different angle (proof element from script) |
| Content reference | `.sales/cadences/email-templates.md` Touch-2 email — different subject line variant from Touch 1 |
| Citation | `[SalesLoft Big Book of Cadences — second email at Day 3 maximises subject-line A/B test signal, tier: scaffolding]` |
| Exit if | Hard bounce or unsubscribe → mark Lost |

### Touch 3 — Day 5 — LinkedIn connect request

| Field | Value |
|---|---|
| Day | 5 |
| Channel | LinkedIn (free InMail / connect request, no Sales Navigator required for v0.1) |
| Goal | Open the social channel; warm the contact for next call |
| Content reference | LinkedIn note ≤300 chars: name + one-sentence reason + no pitch |
| Citation | `[LinkedIn State of Sales — connect requests with personalised note have 2.3x higher acceptance, tier: evidence]` |
| Exit if | Connect request rejected and prospect is decision-maker → continue (LinkedIn is supplementary, not blocking) |

### Touch 4 — Day 7 — Call + voicemail

| Field | Value |
|---|---|
| Day | 7 |
| Channel | Phone with intentional voicemail |
| Goal | First voicemail of the cadence |
| Content reference | `.sales/scripts/call-script-v<latest>.md` §9 Voicemail variant |
| Citation | `[Jeb Blount Fanatical Prospecting voicemail framework, tier: scaffolding]` + `[Gong voicemail data — Tue/Wed end-of-day voicemails have 1.4x higher callback rate, tier: evidence]` |
| Exit if | Live answer → run full script |

### Touch 5 — Day 10 — Email (different angle)

| Field | Value |
|---|---|
| Day | 10 |
| Channel | Email |
| Goal | Reframe — if Touches 1-4 led with the value statement, lead with the prospect's likely pain instead |
| Content reference | `.sales/cadences/email-templates.md` Touch-5 email tagged Josh Braun "Poke the Bear" framework |
| Citation | `[Josh Braun "Poke the Bear" cold-call script + email companion, tier: scaffolding]` |
| Exit if | Reply received → respond inside 1 hour, advance to manual conversation |

### Touch 6 — Day 14 — Call

| Field | Value |
|---|---|
| Day | 14 |
| Channel | Phone |
| Goal | Re-attempt the live conversation; reference the email send if voicemail |
| Content reference | `.sales/scripts/call-script-v<latest>.md` opener variant + "I left you a voicemail last week and an email yesterday — I want to be respectful of your time, do you have 60 seconds?" |
| Citation | `[Voss calibrated question — "do you have 60 seconds?" gives prospect agency, tier: scaffolding]` |

### Touch 7 — Day 18 — LinkedIn message

| Field | Value |
|---|---|
| Day | 18 |
| Channel | LinkedIn DM (only if Touch 3 connect was accepted) |
| Goal | Light, no-pitch nudge |
| Content reference | "Saw your post about <X>" or "Noticed your team is hiring for <Y> — relevant to what we're building" |
| Citation | `[LinkedIn State of Sales — value-add DMs have 4x reply rate over pitch DMs, tier: evidence]` |

### Touch 8 — Day 21 — Break-up email

| Field | Value |
|---|---|
| Day | 21 |
| Channel | Email |
| Goal | Force a binary response: yes / no / not now |
| Content reference | `.sales/cadences/email-templates.md` Touch-8 break-up — short, polite, asks for one-line answer |
| Citation | `[Predictable Revenue / Aaron Ross break-up email convention, tier: scaffolding]` + `[SalesLoft Big Book — break-up emails recover 12-19% of dormant prospects, tier: evidence]` |
| Exit if | Reply received → respond; no reply → continue to Touch 9 |

### Touch 9 — Day 28 — Call (re-attempt after break-up)

| Field | Value |
|---|---|
| Day | 28 |
| Channel | Phone |
| Goal | One more live attempt after the break-up signal |
| Content reference | `.sales/scripts/call-script-v<latest>.md` "I sent you a break-up note last week and figured I owed you one more call" opener |
| Citation | `[Jeb Blount refusal-to-quit doctrine, tier: scaffolding]` |

### Touch 10 — Day 35 — Final break-up email

| Field | Value |
|---|---|
| Day | 35 |
| Channel | Email |
| Goal | Permanent close — "I'll stop reaching out" with a single re-engagement door |
| Content reference | `.sales/cadences/email-templates.md` Touch-10 final email |
| Citation | `[Outreach.io decay curve research — touches beyond ~10 have <2% incremental reply rate, tier: evidence]` |
| Exit | After Touch 10, prospect moves to nurture (re-entry after 90 days) |

## Touches 11-12 — optional retry layer

For high-value prospects (lead score above the threshold defined in `.sales/list-strategy.md`), append two more touches:

- **Touch 11 — Day 45 — Voicemail referencing seasonal trigger** (Spring service push, end-of-quarter, etc.). Citation: `[Sobczak Smart Calling — pre-call trigger research lifts connect rate, tier: scaffolding]`.
- **Touch 12 — Day 60 — Email with a new proof element** (case study from a sign-up since Touch 1). Citation: `[Cialdini Social Proof, tier: vocabulary-only]`.

Beyond Touch 12, return on additional touches drops below 1% per Outreach.io decay-curve research. Do not extend further; route to nurture.

## Channel mix summary

| Channel | Touch count in default cadence | % of total |
|---|---|---|
| Phone (live attempt) | 4 (Touches 1, 6, 9, optional 11 voicemail) | 33-40% |
| Voicemail | 2 (Touches 4, optional 11) | 17-20% |
| Email | 5 (Touches 1, 2, 5, 8, 10, optional 12) | 42-50% |
| LinkedIn | 2 (Touches 3, 7) | 17% |

Total channels: 4. Meets the ≥3-channel target from spec §11.

## Per-touch expected metrics

| Touch | Expected reply / connect rate | Tier |
|---|---|---|
| 1 (call) | 15-25% live connect | evidence |
| 1 (email) | 8-12% reply | evidence |
| 2 (email) | 4-6% reply | evidence |
| 3 (LinkedIn connect) | 25-40% acceptance | evidence |
| 4 (call + VM) | 12-18% live connect, 5-8% callback from VM | evidence |
| 5 (email different angle) | 3-5% reply | scaffolding |
| 6 (call) | 8-12% live connect | scaffolding |
| 7 (LinkedIn DM) | 6-10% reply | evidence |
| 8 (break-up email) | 12-19% reply | evidence |
| 9 (call after break-up) | 6-10% live connect | scaffolding |
| 10 (final break-up) | 3-5% reply | scaffolding |

Reply rates are cumulative — most cadences see 25-40% of prospects engage somewhere across the 10 touches. The 80% / 5+ touches statistic (SalesLoft / IRC Sales Solutions, `tier: evidence`) means abandoning at Touch 3 leaves 60-80% of winnable prospects unworked.

## Adjustments per ICP

- **Owner-operator tradesperson (elocal_clone first user)** — phone-heavy. Move Touches 5 and 7 to phone instead of LinkedIn (most plumbers don't check LinkedIn). Replace LinkedIn touches with text-message touches if the user has explicit consent (TCPA/CASL — never cold-text).
- **Office-based decision-maker (SaaS, professional services)** — email-heavy. The default 12-touch sequence is well-tuned.
- **Enterprise / multi-thread** — out of scope for v0.1. The `accounts` plugin (future) will handle multi-thread cadences.
