# Verification Contract

Create `orchestrator/verification.md` as the reviewer's canonical checklist.

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
