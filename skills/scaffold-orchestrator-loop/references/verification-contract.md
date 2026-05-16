# Verification Contract

Create `orchestrator/roadmaps/<roadmap_id>/<roadmap_revision>/verification.md`
as the repo- and roadmap-specific check list for the active roadmap bundle.
Universal reviewer duties stay in `orchestrator/roles/reviewer.md`.
Repo-wide invariants stay in `orchestrator/project-contract.md`.
Active roadmap bundle semantics stay in
`orchestrator/active-roadmap-bundle.md`.
Shared role obligations stay in `orchestrator/role-contract.md`.

## Required Sections

Use the section set required by
`orchestrator/active-roadmap-bundle.md`. This reference owns how to fill those
sections during scaffold; the repo-local active roadmap bundle contract owns
the exact required section names.

## Baseline Checks

Fill these with repo-specific commands discovered during setup:

- tests
- lint or formatting checks
- build or typecheck checks
- documentation or example checks, if they are part of normal repo quality

If the repo has no automation yet, say so explicitly and give the reviewer the minimum manual checks they must record.

## Alignment Checks

Fill these with checks that prove the approved alignment, especially success
criteria and non-goals that are not covered by baseline automation. Each check
should explain which approved criterion it protects.

Do not restate reviewer role behavior, schema fields, closeout semantics,
worker fan-out rules, or roadmap-update state in `verification.md`. Point
reviewers to:

- `orchestrator/roles/reviewer.md` for approval duties and evidence;
- `orchestrator/selection-record-schema.md` for selection lineage;
- `orchestrator/round-plan-record-schema.md` for worker fan-out checks;
- `orchestrator/round-finalization-schema.md` for review and closeout records;
- `orchestrator/roadmap-update-schema.md` for semantic roadmap updates; and
- `orchestrator/active-roadmap-bundle.md` for status-only versus semantic
  roadmap-update classification.

Use `verification.md` for repo- or roadmap-specific checks only, plus setup
checks such as preserving prior roadmap families during `next-family` and
stopping after the scaffold checkpoint without starting runtime rounds.

## Task-Specific Checks

Tell reviewers to add checks that are specific to the current round, such as:

- new focused tests
- migration verification
- manual UI validation
- fixture or example updates

## Manual Checks

Add manual review steps only when automation cannot cover required behavior.
Keep them concrete enough that a reviewer can record pass/fail evidence.

## Roadmap Overrides

Use this section only for verification rules specific to the active roadmap
revision. Put universal approval criteria and review-record format in
`orchestrator/roles/reviewer.md`, and put shared event schema, fixture,
dry-run, or package-boundary invariants in `orchestrator/project-contract.md`
instead of every roadmap verification file.

Roadmap-specific retry overrides follow
`orchestrator/active-roadmap-bundle.md`. Otherwise record `none`.
