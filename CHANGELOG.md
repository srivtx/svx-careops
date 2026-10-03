# Changelog

All notable changes to this project are documented here. Format follows
[Keep a Changelog](https://keepachangelog.com/); versioning is
[Semantic](https://semver.org/) and deliberately conservative — see
AGENT-GOALS.md standing constraints.

## [Unreleased]

### Changed
- Goal 0 resolved (2026-10-04): Gate 1 (AlayaCare API depth) PASSED —
  397 documented endpoints incl. EVV records, visit/task/schedule CRUD,
  and the write paths the approval loop needs. Gate 2 (absorption)
  FIRES partially — AlayaCare announced agentic AI / AI Form Assistant /
  Clinical Agent (Mar–May 2026); absorption clock now runs on both
  platforms; cross-platform adapter promoted to a day-one design
  requirement. Full endpoint inventory: `docs/gate-log.md`. No version
  bump (docs).

## [0.0.1] — 2026-09-30

### Added
- Task brief only, no code: README (product brief + necessity case +
  verification status + pricing posture), AGENT-GOALS.md (work orders:
  Goal 0 build gates — AlayaCare API depth + absorption check; Goal 1
  EVV-exception triage MVP; Goal 2 write path + billing-ready notes,
  gated; Goal 3 channel build; Goal 4 minor improvements), AGENTS.md
  (mechanics), brief-guard CI.
- Product born from SVX gap-registry row 14, verified OPEN (narrowing,
  absorption clock started) by three kill-search passes (R9, R13b, V2 —
  2026-09-30).
