# Verification Contract

Create `orchestrator/roadmaps/<roadmap_id>/<roadmap_revision>/verification.md` as the reviewer's canonical checklist for the active roadmap bundle.

## Required Sections

- `## Baseline Checks`
- `## Task-Specific Checks`
- `## Approval Criteria`
- `## Reviewer Record Format`

## Baseline Checks

Fill these with repo-specific commands discovered during setup:

- tests
- lint or formatting checks
- build or typecheck checks
- documentation or example checks, if they are part of normal repo quality

If the repo has no automation yet, say so explicitly and give the reviewer the minimum manual checks they must record.

The verification contract should also require reviewers to confirm that:

- the round stayed within the active roadmap bundle recorded in `state.json`;
- the round's recorded `roadmap_id` matches the active family's scaffolded `YYYY-MM-DD-NN-<slug>` identifier rather than a recomputed title-derived value;
- `selection.md` records `roadmap_id`, `roadmap_revision`, and `roadmap_dir`; and
- `review-record.json` records the same roadmap identity when the round finalizes.

## Task-Specific Checks

Tell reviewers to add checks that are specific to the current round, such as:

- new focused tests
- migration verification
- manual UI validation
- fixture or example updates

## Approval Criteria

Approval should require all of the following:

- baseline checks pass
- task-specific checks pass
- reviewer records evidence in `review.md`
- no unresolved blocking issue remains

## Reviewer Record Format

Require a concise record with:

- commands run
- pass or fail result
- evidence summary
- approve or reject decision
