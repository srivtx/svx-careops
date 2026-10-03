# careops — the back-office agent for home-care agencies

**svx-careops** is a read-only agent layer that sits on top of the
home-care platforms agencies already use (AlayaCare first, AxisCare
second). It ingests EVV, visit, and schedule data continuously, finds
the exceptions that today consume a full-time human (missed clock-ins,
EVV mismatches, schedule gaps, incomplete billing notes), and drafts
the fixes — queued for **one-person approval**. It drafts; a human
approves. The agency keeps its platform, keeps its audit trail, and
stops paying for invisible labor.

> **Status: pre-build, verified open with an absorption clock
> running on both major platforms (re-verified 2026-10-04).** This
> repository is a complete, self-contained work order. No product code
> exists yet. Read [`AGENT-GOALS.md`](AGENT-GOALS.md) first — it is the
> task brief; this README is the context.

## Why this is necessary

The SVX research system ran this gap through three kill-search passes —
R9 (19 queries), R13b, and V2 (2026-09-30) — looking for the existing
solver. None was found for the core wedge. The evidence case:

1. **The workload the software leaves behind is a priced full-time
   job.** Agencies pay **$700–$1,000/month per offshore
   AlayaCare-literate virtual assistant** (onlinejobs.ph, Mar 7 2026),
   and a whole BPO vertical — "Remote AlayaCare Outsourcing," staffed
   "inside the agency's own AlayaCare environment" (staffingly.com) —
   exists to operate the agencies' own software. When customers hire
   humans to run your category's products, the back office *is* the
   product gap.
2. **The exception stream never stops regenerating.** Caregiver
   turnover is ~75.5%/yr, flat for years (Activated Insights via
   HHAeXchange, Jul 29 2026; PHI: ~75% in 2024; HCAOA: 77–79%
   2022–2023), at ~$2,600 per departure (PHI via EngineHire, Mar 25
   2026). A permanently churning workforce guarantees missed
   check-ins, schedule holes, and EVV exceptions — forever. This is a
   recurring-revenue machine hiding inside a compliance workflow.
3. **Buyer ≠ user in its purest documented form.** Agency owners buy
   "audit readiness" (AlayaCare is "praised for audit readiness" —
   Capterra reviews); caregivers and office staff suffer the data
   entry (clock-in failures, portal password complaints). The owner
   buys compliance confidence; the office pays in labor. A tool that
   removes the labor *while strengthening* the audit trail sells to
   both sides of that split.
4. **The integration pattern is proven in production.** A non-AI
   third-party product already imports caregiver records out of
   AxisCare via its customer API in production (support page, Sep 3
   2026); AxisCare's API is vendor-confirmed ("available for customers
   connecting external systems"). The technical core is demonstrated
   by a smaller-scope product.
5. **AI is selling in exactly this market, one workflow at a time.**
   AI intake software claims post-call admin cut "from 30 minutes to
   under 5" (sagecare.ai, Mar 25 2026). The single-workflow overlay
   shape works commercially here — and the two biggest back-office
   workflows (EVV exceptions, billing notes) are unclaimed.
6. **Two of five major platforms are already closed to overlays**
   (Axxess: CEHRT patient-access endpoints only; Sandata: EVV API
   serves states and EVV vendors, graded F — supergood.ai API Report
   Card). Platform choice is load-bearing; this repo's first ship is
   chosen accordingly.

The one-sentence necessity argument: **agencies already pay
human-outsourcers $700–$1,000/mo to fix the exceptions their software
generates, the exception stream regenerates itself annually through
75% caregiver churn, and nobody sells the agent that does the drafting
— the incumbent is racing to absorb it, which is itself the strongest
possible validation of the demand.**

## What it is — and is not

- **Is:** a read-only integration + exception triage + **draft**
  queue. Every outgoing change is a draft a human approves with one
  click. The audit trail is the feature: who approved what, when,
  against which source record.
- **Is not:** a platform replacement (agencies won't switch — killed
  in research: Careswitch is switch-shaped and ~$100K-funded), an EVV
  compliance tool (bundled everywhere — killed), an AI visit-note
  scribe for skilled home health (Axxess absorbed it — killed), or an
  autonomous agent that writes to systems of record without approval.
- **Not legal advice, not a medical device:** drafts are
  administrative corrections, not care decisions. The human approver
  owns every change.

## Verification status and build gates

The gap was **OPEN — narrowing, absorption clock started — at
medium-high confidence** as of 2026-09-30 (V2), and **re-verified
2026-10-04 with the Goal-0 gates resolved** (see
[`docs/gate-log.md`](docs/gate-log.md)):

- **The API gate passed decisively:** developer.alayacare.com
  documents **397 endpoints** — EVV records and visit-verification,
  full visit/task/schedule CRUD, caregiver records, progress notes,
  and billing items — including the write paths the approval loop
  needs (`post_visits-{id}-notes`, `patch_visits-{id}`,
  `put_visits-lock`).
- **The absorption clock now runs on BOTH platforms:** AxisCare
  announced in-house AI (Jun 15 2026), and AlayaCare announced agentic
  AI, an AI Form Assistant, and a Clinical Agent (Mar–May 2026,
  "reclaim 80% of time and costs") — absorbing the adjacent
  clinical-forms layer. The EVV-exception + billing-note lane is not
  confirmed absorbed. **The strategic core re-frames to the slice no
  incumbent can copy: cross-platform neutrality** — one back-office
  agent across AlayaCare/AxisCare/WellSky, with the platform adapter
  as a day-one design requirement.
- First ship unchanged in kind (AlayaCare, EVV-exception triage),
  sharpened in positioning (neutrality + audit trail, not raw
  automation claims).

Full evidence: [V2 report](https://github.com/srivtx/svx-research/blob/main/research/track-reports/V2-row14-careops-killsearch.md),
[R9 report](https://github.com/srivtx/svx-research/blob/main/research/track-reports/R9-elder-care-ops.md),
[registry row 14](https://github.com/srivtx/svx-research/blob/main/docs/gap-registry.md).

## Pricing posture

**Sub-$700/month per agency** — deliberately anchored under the
documented offshore-VA line item ($700–$1,000/mo, onlinejobs.ph). The
pitch is not "AI," it is "one less salary line with a better audit
trail." Channels: HCAOA (state associations), home-care franchise
networks, and — deliberately — the VA/BPO firms themselves as resellers
(sell them the agent that does their grunt work).

## The SVX family

This is product #2 of the SVX research-to-product system:

- [svx-research](https://github.com/srivtx/svx-research) — the research
  brain: gap registry, track reports, raw evidence, loop protocol.
- [svx-evalgate](https://github.com/srivtx/svx-evalgate) — product #1:
  deterministic eval gating in CI.
- [svx-parityrun](https://github.com/srivtx/svx-parityrun) — product
  #3: the legacy differential-validation harness (registry row 13).

License: MIT (see [LICENSE](LICENSE)). Version discipline: 0.x until a
real agency (or agency-shaped pilot) approves real drafts in a real
AlayaCare sandbox; see `AGENT-GOALS.md`.
