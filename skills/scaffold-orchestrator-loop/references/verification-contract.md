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
- `selection.md` records `roadmap_id`, `roadmap_revision`, `roadmap_dir`,
  `milestone_id`, `direction_id`, and `extracted_item_id`;
- `review-record.json` records the same roadmap lineage when the round
  finalizes;
- `roadmap_item_id` appears only when the active roadmap revision is still a
  legacy flat roadmap or when a compatibility mirror is explicitly required;
- when `orchestrator/roadmap.md`, `orchestrator/verification.md`, or
  `orchestrator/retry-subloop.md` exist, they match `roadmap_id`,
  `roadmap_revision`, and `roadmap_dir` in `state.json`;
- a `next-family` setup preserved prior roadmap families and revisions
  unchanged; and
- the setup change stopped after the checkpoint commit without starting runtime
  rounds.

If the round used planner-authored worker fan-out, baseline checks should also
require the reviewer to confirm that:

- `worker-plan.json` exists and matches the integrated extracted round scope;
- worker ownership boundaries were respected; and
- approval is based on the integrated round result rather than isolated worker
  slices.

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
- any next-family pointer refresh or preserved-history check passes
- any `pending-merge` refresh or re-review requirement has been satisfied before
  the round is approved for merge

## Reviewer Record Format

Require a concise record with:

- commands run
- pass or fail result
- evidence summary
- approve or reject decision
