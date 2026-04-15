---
title: "HubSpot Free CRM — Bootstrap Patterns for Solo and 2-Person Teams"
author: Compiled from HubSpot Developer Documentation, Knowledge Base, and Free Tier limitations as of April 2026
year: 2026
type: playbook
tier: scaffolding
url: https://developers.hubspot.com/docs/api/crm/contacts
cited-by-agents: [crm-architect]
---

# HubSpot Free — Bootstrap Patterns

## One-line summary
HubSpot Free is the default CRM choice for the elocal_clone bootstrap context — 2 paid seats, unlimited contacts, working pipeline, REST API access for one-shot setup — but its free tier is missing workflows, sequences, and most automation, so the bootstrap pattern is to provision the pipeline shape and fields via the API once and then drive everything else manually until revenue justifies an upgrade.

## Why this tier
Scaffolding, not evidence. There is no published study showing HubSpot Free outperforms Pipedrive Essential or Close.io for solo founders. What earns it scaffolding-tier is that the **field schema and pipeline-stage discipline are forcing functions**: the bootstrap pattern below makes the founder commit to a specific lifecycle stage definition, a specific lead-score formula, and a specific daily KPI dashboard before any cold call goes out. The shape of the patterns transfers cleanly to Pipedrive or Close if the founder switches CRMs later. The choice of HubSpot Free is justified by (a) the 2-paid-seat fit for founder + wife, (b) the open REST API for `--apply` automation, and (c) the founder's stated preference for the lowest-friction free tier.

## Key concepts
- **The 2-seat ceiling is real.** HubSpot Free includes 2 paid seats and unlimited free seats, but free seats cannot own contacts or be sales reps in the pipeline — they are read-only. For a founder + wife outbound team, both must be paid seats, which fits.
- **No workflows on Free.** HubSpot Free has zero workflow automation, zero sequences, zero email send automation. The pipeline can be moved by hand or via API; everything else is manual until paid Sales Hub Starter ($45/seat/mo).
- **Pipeline stages — recommended bootstrap shape.** Lead → Contacted → Conversation → Qualified → Waitlist → Activated → Won / Lost. Seven stages, each with explicit exit criteria. The Waitlist stage is specific to the elocal_clone two-stage waitlist pattern — contractors who said yes but are waiting for their region/vertical to fill before activation.
- **Lifecycle stages (separate from pipeline).** HubSpot has its own `lifecyclestage` field on Contacts: subscriber → lead → marketingqualifiedlead → salesqualifiedlead → opportunity → customer → evangelist. The bootstrap maps the elocal_clone stages onto these by aliasing — not by overriding — so future integrations work.
- **Custom fields the trades-vertical bootstrap needs.** trade_category (plumber / electrician / HVAC / appliance / other), service_radius_km, current_lead_gen_provider (HomeAdvisor / Angi / HomeStars / none / other), business_size_trucks, founding_member_status (yes / no), recording_played_date, free_calls_used, calls_remaining, owner_phone_pref (mobile / shop / both), best_callback_window, language (EN / FR / both). Twelve fields, all set up via the Properties API in a single bootstrap script.
- **Lead score formula (Free-tier-compatible).** HubSpot Free does not include the predictive lead-scoring AI; the formula is computed in a custom number field. Recommended bootstrap formula: `+10 if currently using HomeAdvisor`, `+5 per truck`, `+15 if Hamilton/GTA/Toronto/Mississauga`, `+10 if recording was played`, `+5 if owner answered direct line`. Recompute on every Contact write via the API.
- **The KPI dashboard.** HubSpot Free Reports allows custom dashboards. The daily-dashboard spec for the founder + wife: dials today, conversations today (>90sec), recordings played today, free-call signups today, week-to-date totals, conversion rates between adjacent funnel stages, top 5 active contacts by lead score. Refresh manually each morning.
- **The `--apply` script pattern.** The plugin's optional `/sales:crm-setup --apply` runs a Python script that uses the HubSpot REST API (api.hubapi.com/crm/v3) with a Private App access token to (1) create the pipeline, (2) define the 7 stages, (3) define the 12 custom fields, (4) seed the dashboard. One-shot, idempotent (uses HubSpot's PATCH semantics so re-running updates rather than duplicates).

## Direct quotes (fair use, attributed)

1. "Properties are the fields that store information about your CRM records. HubSpot includes default properties, but you can also create custom properties to capture data specific to your business." — HubSpot Developer Documentation, https://developers.hubspot.com/docs/api/crm/properties (verify exact wording on refresh).

2. "The Free tools include access to up to 2 paid seats with sales tools, the contact records database, deal pipelines, tasks and activities, email tracking and notifications, and reporting dashboards. Workflow automation requires Sales Hub Starter or higher." — paraphrase of HubSpot pricing page, hubspot.com/pricing/sales (verify on refresh; HubSpot updates the free-tier feature list periodically).

*(Both quotes are paraphrased from HubSpot's own documentation, which is regularly updated. The crm-architect agent should verify on each `/sales:crm-setup` run.)*

## When to cite
Cited by `crm-architect` for every HubSpot Free recommendation — pipeline stages, custom field list, lead-score formula, dashboard spec, and `--apply` script generation. The crm-architect should also cite this file when comparing HubSpot Free vs Pipedrive Essential vs Close.io and recommending HubSpot Free as the default for a 2-person trades-outbound team.

## Anti-citation guidance
Do not cite this file as evidence that HubSpot Free is the BEST CRM — it is the recommended default for the specific elocal_clone context (2 paid seats, REST API access, no workflow needed at this stage). Pipedrive Essential ($14/seat/mo) is a defensible alternative if the founder wants paid workflow automation from day 1, and Close.io is the better choice for high-volume dialing teams. Do not generate the `--apply` Python script with a hardcoded API key — read the key from `.sales/config/secrets.local.md` (gitignored) or from environment variable `HUBSPOT_PRIVATE_APP_TOKEN`. Refresh HubSpot's Free tier feature list on every `/sales:crm-setup` run since the limits change quarterly.
