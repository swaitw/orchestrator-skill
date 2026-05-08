# Verification Contract

Create `orchestrator/roadmaps/<roadmap_id>/<roadmap_revision>/verification.md`
as the repo- and roadmap-specific check list for the active roadmap bundle.
Universal reviewer duties stay in `orchestrator/roles/reviewer.md`.
Repo-wide invariants stay in `orchestrator/project-contract.md`.

## Required Sections

- `## Baseline Checks`
- `## Alignment Checks`
- `## Task-Specific Checks`
- `## Manual Checks`
- `## Roadmap Overrides`

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

Do not restate the full reviewer role in `verification.md`. Keep universal
review expectations in shared role or setup-contract text, including:

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

For planner-authored worker fan-out, shared review expectations should also
cover:

- `worker-plan.json` exists, conforms to `orchestrator/worker-plan-schema.md`,
  and matches the integrated extracted round scope;
- worker ownership boundaries were respected; and
- approval is based on the integrated round result rather than isolated worker
  slices.

For roadmap updates, shared review expectations should cover:

- `roadmap-update.md` exists under `orchestrator/roadmap-updates/`;
- the update is justified by the merged round evidence;
- used roadmap revisions remain immutable unless a new revision is authored;
- `state.json.roadmap_update` points at the active update branch, worktree, and
  artifacts while the update is in progress; and
- new `roadmap_id`, `roadmap_revision`, or `roadmap_dir` metadata is activated
  only after `roadmap-update-review.md` approves the change.

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
