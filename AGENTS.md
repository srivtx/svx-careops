# AGENTS.md — agent context for svx-careops

**What this repo is.** The pre-build product repo for SVX gap-registry
row 14 — a read-only, human-approves-everything agent layer that
drafts EVV-exception fixes and billing-ready notes on top of home-care
platforms (AlayaCare first). The complete work orders live in
[`AGENT-GOALS.md`](AGENT-GOALS.md); the evidence case and verification
status live in the [README](README.md). Both were written 2026-09-30
from three kill-search passes (R9, R13b, V2) in the
[research repo](https://github.com/srivtx/svx-research) — the whole
story of how this repo came to exist is that repo's `AGENT-MISSION.md`.

## The four facts that survive any context switch

1. **Read-only + human approval is the product**, not a safety
   concession — the buyer pays for audit readiness, the user pays in
   labor, and the one-person approval queue is what serves both.
2. **First platform is AlayaCare, not AxisCare** — even though
   AxisCare's API is the best-evidenced, its vendor put AI automation
   on its own 2026 roadmap (Jun 15 2026). We build where the
   workaround market is thickest and the incumbent is not racing us.
3. **Pricing is sub-$700/mo** — anchored under the documented offshore
   VA line item ($700–$1,000/mo). The pitch is "one less salary line
   with a better audit trail."
4. **Do not build intake, do not build scheduling optimization, do
   not build a platform** — occupied (sagecare.ai), incumbent-roadmap
   (AxisCare), and killed territory (Careswitch) respectively.

## Working rules

- **Tests-first.** Fixtures (synthetic platform snapshots) are the
  test substrate; CI never touches a live API. GitHub Actions green
  from the first commit.
- **Python 3.10+, std-lib only at runtime** (the family rule from
  svx-evalgate); ruff + mypy clean; determinism where testable — same
  snapshot + same rules version → identical exception set.
- **PHI discipline before features.** `docs/data-handling.md` must
  exist and be honest before any external call ships. Local-first,
  agency-owned credentials, zero-retention LLM settings at most, and
  prefer no-LLM implementations when they cover the case. Home-care
  data is health data; treat it that way in every design decision.
- **Versioning is conservative and owner-gated.** 0.x until real
  drafts are approved in a real agency sandbox; docs never bump.
- **Evidence discipline.** Market claims cite the research reports
  (R9/V2) or dated pilot facts. No invented numbers, no compliance
  claims beyond what the audit log actually proves.
- **Gates are gates.** Goal 0 (API depth + absorption check) runs
  before code. Goal 2 waits for Goal 1 + the write-path outcome.
  Gate outcomes go to `docs/gate-log.md`, dated, linked.
- **Kill-list respect.** If AlayaCare ships native AI back-office
  features, or a funded overlay takes the lane, record it honestly
  in gate-log + README verification status and notify the maintainer.
  A killed gap is a successful finding in this system.
- **Worklog protocol.** Substantial sessions append to
  `/home/z/my-project/worklog.md` (research-side) and `CHANGELOG.md`
  under `[Unreleased]` (this repo). Commit messages: imperative, one
  line.

## Repo map (as of the task-brief commit)

```
README.md           # product brief + necessity case + verification status
AGENT-GOALS.md      # THE TASK BRIEF — work orders, constraints, gates
AGENTS.md           # this file — mechanics for any agent
CHANGELOG.md        # 0.0.1 = task brief only, no code
LICENSE             # MIT
.github/workflows/brief-guard.yml   # CI: validates the brief stays intact
docs/               # gate-log.md, data-handling.md land here
```
