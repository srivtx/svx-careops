# Gate Log — careops

Build gates and market-structure findings, newest first. Per
AGENT-GOALS.md, gates are resolved before code; outcomes recorded here,
dated, with sources.

---

## 2026-10-04 — Goal 0 resolved: AlayaCare API depth (Gate 1) and the absorption check (Gate 2)

**Method:** direct reads (owner session, 2026-10-04) — AlayaCare's
developer portal read directly (no search), plus one search for the
absorption check. Raw captures in the research repo at
`research/raw-search-results/w4-direct/`.

### Gate 1 — AlayaCare API endpoint depth: **PASSED, decisively**

`developer.alayacare.com` (the new documentation platform; the old one
lives at alayacare.github.io/external-integration-docs) documents
**397 reference endpoints**. The wedge's five required categories are
all present, plus the write paths:

| careops need | Confirmed endpoints (verbatim names) |
|---|---|
| EVV records | `get_evv-visits`, `get_visit-verification-visits-{id}`, `get_evv-exports` (+latest-export-date), `post_evv-exports`, `put_evv-exports-{id}` |
| Visits / shifts (read) | `get_visits`, `get_visits-by-id-visit-id`, `get_visits-{id}-accounting`, `get_facility-visits`, `get_visits-client-id-next-visit-id` |
| Visits (write — the approval path) | `post_visits`, `put_visits`, `patch_visits-{id}`, **`post_visits-{id}-notes`**, `put_visits-lock` (lock-after-approval!), `put_visits-{id}-interventions` |
| Schedules / tasks | `get_tasks` + full task CRUD (`createtask`, `updatetask`, `changetaskstatus`, `addtaskcomment`), `get_services`, `get_availability-types`, `put_employees-{id}-unavailabilities` |
| Caregiver records | `get_employees` (+by external id), skills, `get_employees-{id}-unavailabilities`, `get_profile-employee` |
| Notes / clinical | `get_progress-notes-client-id`, `post_progress-notes-{id}`, care-provider notes CRUD, `get_forms`, CMS-485 flows |
| Billing | `getbillableitems`, `createbillableitem`, `updatebillableitem`, `setbillableitemsstatus`, `regeneratebillableitems`, billing cycles/invoices, transactions |

Additional confirmations: authentication + rate-limiting docs exist;
Flat File and SQS-queue integration paths exist; API Terms PDF is
current (uploaded Feb 2026). Portal states scope plainly: clients,
employees, scheduling (visits/tasks/care plans), clinical data, files,
accounting, medications.

**Implication for Goal 1:** the write gate is GREEN — the
approve-and-apply loop (`careops approve` → `post_visits-{id}-notes`,
`patch_visits-{id}`, `put_visits-lock`) is supported end to end.
Goal 1's "NARROWED if no write paths" contingency does not apply.

### Gate 2 — Absorption check: **FIRES — partial**

AlayaCare — chosen as first platform precisely because V2 found "no
in-house AI absorption signal" — has since announced a major AI push:

- Toronto Star press release (March 2026): "AlayaCare empowers home
  care agencies to **reclaim 80% of time and costs with new AI
  automation**."
- Yahoo Finance (May 7 2026): "AlayaCare **unveils agentic AI**…"
- leadiq.com: March 2026 "major platform update featuring **AI-powered
  autonomous agents**."
- lyv-ia.com (Sep 20 2026, competitor analysis): AlayaCare lists an
  **AI Form Assistant** and a **Clinical Agent** among its features.
- Context: AlayaCare Connector (code-free integration tool) also
  shipped — the platform is opening up as it absorbs AI.

**What this does and does not absorb:** the announced capabilities are
form-filling / clinical-documentation shaped ("Form Assistant",
"Clinical Agent") — the Axxess-Care-2.0 wedge, adjacent to but not
confirmed on careops's first-ship lane (EVV-exception fixes +
billing-ready notes). The exact EVV-exception/billing-note coverage was
not readable (AlayaCare's blog is challenge-walled; the announcements
are paywalled/summarized in secondary sources). **The absorption
clock is now running on BOTH major platforms** (AxisCare Jun 15 2026;
AlayaCare Mar–May 2026).

**Re-frame (per AGENT-GOALS Goal 0 gate 2 protocol — maintainer
notified via this log and the CHANGELOG):**

1. **The cross-platform slice is now the strategic core.** No
   incumbent will ever be neutral across AlayaCare + AxisCare +
   WellSky; careops's value proposition "your agency's back office,
   whichever platform you're on — and when you switch" is the slice
   neither incumbent can copy. Multi-platform neutrality moves from a
   Goal 4 afterthought to a Goal 1-2 design requirement (platform
   adapter layer from day one).
2. **The exception lane (EVV mismatches + billing notes) remains the
   first ship** — incumbents are announcing clinical-forms AI, not
   exception triage; but the window is no longer years, it is quarters.
3. **Pricing pressure:** AlayaCare's "reclaim 80% of time and costs"
   marketing will reset buyer expectations — careops's pitch must
   compete on cross-platform neutrality + audit trail, not raw
   automation claims.

### Gate 3 — Re-verify trigger sweep

AxisCare AI: no GA evidence beyond the Jun 15 2026 announcement
(unchanged). sagecare.ai: still intake-shaped. No EVV-exception or
billing-note AI product surfaced anywhere.

**Bottom line: Goal 1 proceeds — AlayaCare first, write paths
confirmed — with the cross-platform adapter as a first-class design
requirement and the absorption clock explicitly acknowledged in all
positioning.**

---

## Next scheduled gates

- Before any external pilot: re-run the absorption check (AlayaCare
  Form Assistant / Clinical Agent GA status; any EVV/billing AI from
  either platform).
- WellSky PC API depth (second-platform feasibility) — read vendor
  docs directly; still unverified after 429-blocked search passes.
