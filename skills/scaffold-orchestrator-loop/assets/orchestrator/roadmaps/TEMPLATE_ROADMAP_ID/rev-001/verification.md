# Verification Checks

Roadmap revision: `orchestrator/roadmaps/TEMPLATE_ROADMAP_ID/rev-001/`.
Tailor this checklist during setup.

This file records repo- and roadmap-specific checks. Universal reviewer duties,
lineage requirements, evidence requirements, and approve/reject format live in
`orchestrator/roles/reviewer.md`.
Repo-wide invariants live in `orchestrator/project-contract.md`; reference them
here only when this roadmap needs a specific check or override.
Active roadmap bundle semantics live in
`orchestrator/active-roadmap-bundle.md`.
Terminal detection, milestone ids, direction ids, and status-only closeout
anchors live in `roadmap-view.json`.
Reviewer-approved status-only round closeout must be recorded in
`review-record.json`; semantic roadmap changes use `update-roadmap`.

## Baseline Checks

- Command:
  Why:

## Alignment Checks

- Criterion:
  Check:

## Task-Specific Checks

- Add round-specific checks here before reviewer approval.

## Manual Checks

- Add manual review steps here when the repo has no reliable automation for a
  required behavior.

## Roadmap Overrides

- none

Use this section for roadmap-specific retry policy only when the active
revision needs behavior beyond the shared runtime retry mechanics. Otherwise
leave it as `none`.
