# AGENT-GOALS.md — careops work orders

**Self-contained work orders for svx-careops's open goals.** Any agent
(human or AI) should be able to pick one goal from this file and execute
it with no other context than this document plus
[`AGENTS.md`](AGENTS.md) and the [README](README.md). The research that
justified this product lives in the
[research repo](https://github.com/srivtx/svx-research) — track reports
R9, R13b, and V2 — but everything needed to *build* is here. This is
the task brief the product was born from.

**Claiming a goal:** read it fully, check the repo state hasn't already
shipped it, re-run the gates if the goal says so, do the work, add a
`CHANGELOG.md` entry under `[Unreleased]`, and move the goal to the
"Shipped" section at the bottom with its commit SHA.

## Standing constraints (apply to every goal)

1. **Read-only by default.** The integration never writes to the
   platform API without an explicit human approval event. Drafts,
   diffs, and approval provenance are stored locally (agency-side) and
   exportable — the audit trail is a *feature*, not a byproduct.
2. **Human-in-the-loop is the product, not a limitation.** The buyer's
   word is "audit readiness"; the user's word is "less grunt work."
   One-person approval delivers both. Autonomy creep is a product
   failure, not an upgrade.
3. **No platform replacement, ever.** careops overlays AlayaCare /
   AxisCare / (future) WellSky PC. Replacement-shaped ideas are killed
   territory (Careswitch) and violate the thesis.
4. **PHI discipline.** Home-care data is health data: local-first
   storage, credentials in the agency's own secret store, no
   third-party analytics on client/caregiver records, no training on
   agency data, and any LLM use runs with zero-retention settings at
   most — document exactly what leaves the machine in
   `docs/data-handling.md` before the first external call. If the
   constraint can be met with no LLM at all, it must be.
5. **Pricing posture is product spec:** sub-$700/mo per agency
   (anchored under the documented VA line item). Features that price
   the tool above a salary-line replacement are out of scope.
6. **Versioning discipline:** 0.x until real drafts are approved in a
   real agency sandbox. Conservative bumps, owner-gated majors, never
   a bump for docs.
7. **Evidence discipline:** market claims cite the research reports;
   product claims cite checked-in artifacts or pilot facts. No
   invented benchmarks, no invented compliance claims.

---

## Goal 0 — Build gates (RESOLVED 2026-10-04 — see `docs/gate-log.md`)

**Outcome: Gate 1 (API depth) PASSED decisively — 397 documented
endpoints at developer.alayacare.com, all five read categories plus
the write paths (`post_visits-{id}-notes`, `patch_visits-{id}`,
`put_visits-lock`). Gate 2 (absorption) FIRES partially — AlayaCare
announced agentic AI / AI Form Assistant / Clinical Agent (Mar–May
2026, secondary sources), absorbing the adjacent clinical-forms
layer; the EVV-exception + billing-note lane is not confirmed
absorbed, but the clock runs on both platforms now. **Goal 1
proceeds, re-framed: the cross-platform adapter layer is a
first-class design requirement from day one, and all positioning
acknowledges the absorption clock.** Full evidence and endpoint
inventory: [`docs/gate-log.md`](docs/gate-log.md).

**Original work order (kept for the record):**

**Why this goal exists.** The gap is OPEN at medium-high confidence,
not high, and it has an **absorption clock** — the incumbent's own 2026
AI roadmap (AxisCare, Jun 15 2026) is the single most likely thing to
close it. Gates convert the two open threads into facts before
engineering starts.

**Work order:**

1. **AlayaCare API endpoint-depth pass** — the technical gate. Read
   AlayaCare's developer documentation *directly* (browser/API, not
   web search — endpoint queries failed 429/junk in two passes).
   Record in `docs/gate-log.md`: auth model, rate limits, and the
   concrete endpoints for EVV records, visits/shifts, schedules,
   caregiver/client records, and notes. The wedge needs **read** on
   all five and **write on EVV-exception/visit-note update paths**.
   - If write paths don't exist: the product narrows to
     detect + draft-for-manual-entry (still viable — the VA does it
     by hand today — but slower ROI); record and re-frame Goal 1's
     approval queue accordingly.
2. **Absorption check:** AlayaCare's blog/product pages and news for
   any native AI back-office/automation announcement since 2026-09-30.
   A GA native feature set on the first-ship workflow = stop and
   notify the maintainer before building.
3. **Re-verify trigger sweep** (same pass): AxisCare AI features
   reaching GA; any EVV-exception or billing-note AI product
   surfacing anywhere. Findings → `docs/gate-log.md`, dated, linked.

**Acceptance criteria:** `docs/gate-log.md` exists with all three gates
resolved (or explicitly re-framed), dated, sources linked. If a gate
fires hard (AlayaCare GA absorption of the wedge), the README's
verification status is updated in the same commit and the maintainer is
notified in the CHANGELOG entry.

---

## Goal 1 — The first ship: AlayaCare EVV-exception triage

**Why this goal exists.** The exception stream is the priced workload
($700–$1,000/mo VA) and EVV is its densest source — missed clock-ins,
GPS/EVV mismatches, schedule-vs-actual conflicts. Triage-only (detect +
explain + draft fix text) is valuable with **zero write access** and is
therefore the honest MVP: it works even if Goal 0's write-path gate
narrows.

**Product shape (the contract):**

```
careops connect alayacare   # OAuth/token, read-only scopes requested first
careops watch               # poll EVV/visit/schedule (respecting rate limits)
careops triage              # exception queue: detect → explain → draft
careops approve             # apply the approved draft (only when write gate is green)
```

- **Detect (deterministic, no LLM):** missed/late clock-in, EVV
  mismatch (scheduled vs actual), schedule gap, unassigned shift,
  duplicate visit, note-missing-after-visit. Rules are data (versioned
  YAML), not code — the same shape as parityrun's tolerance rules.
- **Explain (deterministic first):** every exception row carries the
  source records that triggered it, timestamps, and the rule version.
  The office coordinator should be able to verify any finding against
  the platform UI in under 30 seconds.
- **Draft (this is where LLM assist is allowed, if allowed at all):**
  the correction draft — EVV exception text, adjusted times with
  reason codes, or a billing-ready visit note — citing the source
  records inline. If no-LLM templates cover 80% of cases, ship
  no-LLM first and treat LLM drafting as a later enhancement gated on
  the data-handling doc (constraint #4).
- **Approval queue:** one-person approve/reject/edit; every action
  logged with provenance (who, when, which source records, which rule
  version) in a local, exportable audit log — this is the "audit
  readiness" the buyer pays for.
- **Determinism where testable:** same platform snapshot + same rules
  version → identical exception set and identical draft (when
  deterministic drafting). CI runs against recorded fixture snapshots,
  never a live API.

**Work order (tests-first):**

1. Fixtures: a synthetic AlayaCare-shaped API snapshot (schema from
   Goal 0's endpoint pass) with 3 weeks of visits containing seeded
   exceptions (missed clock-in, EVV mismatch, schedule gap) — the
   test substrate, checked in.
2. Rules engine + detectors with failing-first tests; triage queue
   CLI; then `connect` (sandbox-credentialed) last.
3. Docs: `docs/data-handling.md` (what leaves the machine: ideally
   nothing), `docs/exception-catalog.md` (each rule, its rationale,
   its source in the research), and a walkthrough from the fixtures.
4. CI from the first commit: lint, types, tests, fixture-driven
   end-to-end triage run — no network in CI.

**Acceptance criteria:** fixtures produce the exact seeded exception
set with explanations; drafts for the 3 seeded classes are correct and
provenance-complete; audit log round-trips; zero external calls in
tests; ≥ 40 tests; version 0.1.0 exactly once. If the write gate is
green, `approve` applies one draft end to end in an AlayaCare sandbox
(pilot prerequisite) — else this criterion is marked NARROWED in
gate-log and deferred to Goal 2.

**Explicitly out of scope for Goal 1:** intake (sagecare.ai occupies
it), scheduling optimization (on AxisCare's own 2026 roadmap — head-on
collision), any autonomous write, multi-platform support.

---

## Goal 2 — Write path + billing-ready notes (GATED on Goal 0's write-gate outcome)

**Why this goal exists.** Full ROI is the write: the approved draft
lands in the platform, and the visit-note side generates the
billing-ready documentation that today consumes the VA's day. Only
after Goal 0 confirms write paths and Goal 1 proves triage.

**Work order:**

1. `careops approve` applies drafts via the platform API — one
   approval = one idempotent write, with the approval provenance
   stored locally and (where the API allows) attached to the record.
2. Billing-ready note drafting from visit + EVV + schedule data
   (deterministic templates first; LLM enhancement only per
   constraint #4 and the data-handling doc).
3. Reconciliation sweep: nightly re-read confirms every applied draft
   actually landed (write-audit-read-back) — the feature that makes
   "audit readiness" a testable claim.
4. Pilot playbook: `docs/pilot-runbook.md` — sandbox onboarding,
   credential setup, the 30-day pilot shape, and the before/after
   metrics to record (exceptions/week, approval time, VA hours
   displaced).

**Acceptance criteria:** approve-verify-read-back round-trip in
sandbox; billing-note drafts pass a fixed rubric (complete fields,
reason codes, provenance) checked in tests; runbook reviewed; version
0.2.0 once.

---

## Goal 3 — Channel build: associations, franchises, VA/BPO resellers

**Why this goal exists.** The research named the distribution channels;
this goal operationalizes them. Not a coding goal — a go-to-market
artifact set.

**Work order:**

1. `docs/channel-hcaoa.md` — the state-association pitch (one-pager:
   the exception math, the $700–$1,000/mo anchor, the audit trail).
2. `docs/channel-franchise.md` — franchise-network kit (multi-agency
   onboarding, per-agency tenancy, the "one less salary line" pitch).
3. `docs/channel-bpo.md` — the deliberate counter-intuitive one: sell
   the VA/BPO firms the agent that does their grunt work — margin
   tooling for the exact industry that proves the demand
   (staffingly-class partners).
4. Each channel doc carries its honest kill-check: what evidence
   would say this channel is closed, and who owns checking it.

**Acceptance criteria:** three channel docs, each one page, each
citing the research evidence (R9/V2) for its claims; maintainer
review; no version bump (docs).

---

## Goal 4 — Smaller improvements (pick up anytime)

- ~~AxisCare connector (second platform — the best-evidenced API;
   build only after the AxisCare absorption status is re-checked; its
   vendor is racing its own AI roadmap).~~ **PROMOTED to a Goal 1-2
design requirement by the 2026-10-04 gate outcome** — the platform
adapter layer is now first-class (cross-platform neutrality is the
slice no incumbent will copy); the AxisCare adapter itself still
lands second, after the absorption re-check.
- WellSky PC feasibility note (STILL-UNVERIFIED after two passes —
   read the vendor's API docs directly before any connector work).
- Exception-digest email (weekly owner summary — the "observation
  system" shape from registry gap #6).
- Family-communication and documentation-time research items are
  **retired from search** (3 failed query runs each) — resolve by
  direct product-page/literature reads in the research repo, not here.

---

## Re-verify triggers (any of these → re-run Goal 0 before further work)

- AlayaCare ships native AI back-office / EVV-exception features
  (as of 2026-10-04: agentic AI, Form Assistant, Clinical Agent
  announced Mar–May 2026 — the adjacent layer; see `docs/gate-log.md`).
- AxisCare's 2026 AI automation reaches GA on scheduling/exception
  workflows.
- sagecare.ai (or any overlay) expands from intake into EVV/billing.
- A funded (> $1M) AI-native or overlay competitor surfaces in home
  care back office.

## Shipped goals (append with commit SHA when a goal completes)

| Goal | Shipped in | Notes |
|------|-----------|-------|
| — | — | none yet — Goal 0 is the mandatory first move |
