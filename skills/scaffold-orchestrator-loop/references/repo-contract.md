# Repo Contract

Scaffold or advance a visible top-level `orchestrator/` directory in the target
repository. The required repo contract lives under `orchestrator/`, including
role definitions and worktree preparation.

## Setup Modes

`bootstrap` creates the initial repo-local control plane by copying the full
scaffold tree into a repository that does not yet have top-level
`orchestrator/`.

`next-family` reuses an existing terminal control plane. It must:

- detect `contract_version` and `roadmap_style` before drafting the new family
- reuse existing `orchestrator/roles/`, `orchestrator/rounds/`, and
  `orchestrator/worktrees/`
- scaffold only missing shared control-plane files if the contract is
  incomplete
- create a fresh active bundle under
  `orchestrator/roadmaps/<roadmap_id>/rev-001/`
- preserve `legacy-flat` roadmap structure unless the setup explicitly records
  a migration to `strategy-backlog`; preserved legacy flat roadmaps must follow
  `orchestrator/legacy-flat-roadmap.md`
- preserve prior families, prior revisions, and prior round artifacts as
  immutable history
- refresh `orchestrator/roadmap.md`, `orchestrator/verification.md`, and
  `orchestrator/retry-subloop.md` when those top-level pointer stubs already
  exist

Do not use `next-family` when the existing control plane is still live or the
active roadmap bundle is unfinished under its style-specific parser.

## Required Files

- `orchestrator/state.json`
- `orchestrator/state-schema.md`
- `orchestrator/project-contract.md`
- `orchestrator/legacy-flat-roadmap.md`
- `orchestrator/worker-plan-schema.md`
- `orchestrator/roadmaps/<roadmap_id>/roadmap-history.md`
- `orchestrator/roadmaps/<roadmap_id>/<roadmap_revision>/roadmap.md`
- `orchestrator/roadmaps/<roadmap_id>/<roadmap_revision>/retry-subloop.md`
- `orchestrator/roadmaps/<roadmap_id>/<roadmap_revision>/verification.md`
- `orchestrator/roadmap-updates/`
- `orchestrator/roles/guider.md`
- `orchestrator/roles/planner.md`
- `orchestrator/roles/implementer.md`
- `orchestrator/roles/reviewer.md`
- `orchestrator/roles/merger.md`
- `orchestrator/roles/recovery-investigator.md`
- `orchestrator/rounds/`
- `orchestrator/worktrees/`

Optional compatibility files:

- `orchestrator/roadmap.md`
- `orchestrator/verification.md`
- `orchestrator/retry-subloop.md`

When those pointer files exist, refresh them to match `state.json`. They are
non-authoritative convenience pointers only; runtime resolves the live bundle
from `roadmap_id`, `roadmap_revision`, and `roadmap_dir` in `state.json`.

## `state.json` Schema

Use `orchestrator/state-schema.md` as the canonical field list and legacy
compatibility reference for `orchestrator/state.json`. Do not restate the schema
table in roadmap bundle files or role prompts.

New scaffolds should write `contract_version: "orchestrator-v2"` and
`roadmap_style: "strategy-backlog"`. Missing `contract_version` means legacy
contract compatibility. Missing `roadmap_style` means the repo is a legacy flat
roadmap repo unless an explicit migration records otherwise.

`next-family` reset rules:

- preserve `base_branch`
- preserve `last_completed_round`
- preserve the existing `roadmap_style` when it is `legacy-flat`, unless the
  setup is explicitly performing a recorded migration to `strategy-backlog`
- set the new `roadmap_id`, `roadmap_revision`, and `roadmap_dir`
- set legacy `stage` to `null` because no single round is active yet
- set `controller_stage` to `dispatch-rounds`
- clear `active_round_id`, `active_rounds`, and `pending_merge_rounds`
- clear `roadmap_update`
- clear legacy mirror fields such as `current_task`, `branch`,
  `worktree_path`, `active_round_dir`, and `round_artifacts`
- clear `resume_error`, `resume_errors`, and `retry`

## File Ownership

- The orchestrator may update `orchestrator/state.json` and round bookkeeping only.
- The scaffolded role files under `orchestrator/roles/` are required repo contract files. Tailor them during setup and keep them tracked in the main checkout.
- Approved scaffold alignment lives in the active roadmap bundle,
  `orchestrator/project-contract.md`, and `verification.md`. Do not leave
  strategy, success criteria, or non-goals only in chat history.
- Repo-wide invariants live in `orchestrator/project-contract.md`. Roadmaps and
  role files should point to that file instead of repeating shared event,
  fixture, dry-run, or package-boundary rules.
- Each `project-contract.md` section must contain concrete repo-specific
  entries or the exact phrase `none discovered yet`; blank headings are not a
  valid setup result.
- Delegated role agents author roadmap bundle content, round artifacts, review records, and merge notes.
- The guider owns task selection and roadmap updates.
- Once any round has used a roadmap revision, keep that revision immutable. Publish a new revision directory instead of rewriting a used revision.
- Keep completed roadmap history in
  `orchestrator/roadmaps/<roadmap_id>/roadmap-history.md` or as compact
  completion pointers. Active revisions should not copy all completed item
  bodies forward.
- Do not widen a `next-family` setup into a full shared-contract refresh unless
  the user explicitly asks for that broader change.

## Round Artifacts

Each round folder should contain delegated artifacts only. Start with these names:

- `selection.md`
- `plan.md`
- `implementation-notes.md`
- `review.md`
- `review-record.json`
- `merge.md`

Add more files only when a round needs them.

Every round must record the active `roadmap_id`, `roadmap_revision`,
`roadmap_dir`, `milestone_id`, `direction_id`, and `extracted_item_id` in
`selection.md` and `review-record.json` so archived packets remain
self-contained. Include `roadmap_item_id` only when the active roadmap
revision is still a legacy flat roadmap or when a compatibility mirror is
required for older runtime readers.

For live rounds, artifact paths recorded in `state.json` are repo-relative
paths resolved inside that round's `worktree_path`. The parent checkout sees
those artifacts only after the round branch is merged, so runtime must inspect
the recorded worktree first while a round is active.

When planner-authored worker fan-out is active, the round should also contain:

- `worker-plan.json`
- `workers/<worker-id>/assignment.md`
- `workers/<worker-id>/implementation-notes.md`
- `workers/<worker-id>/handoff.md`

`worker-plan.json` is controller-readable machine state for worker scheduling.
The `workers/` subtree is human-facing evidence owned by delegated workers.
The JSON shape, allowed worker statuses, dependency fields, and integration
lifecycle are defined by `orchestrator/worker-plan-schema.md`.

## Roadmap Update Artifacts

After a successful round merge, `update-roadmap` is a delegated, reviewable
stage. It must use:

- branch `orchestrator/roadmap-update-<round-id>-<slug>`
- worktree `orchestrator/worktrees/roadmap-update-<round-id>`
- artifact `orchestrator/roadmap-updates/<round-id>-roadmap-update.md`
- review artifact
  `orchestrator/roadmap-updates/<round-id>-roadmap-update-review.md`
- `state.json.roadmap_update` record pointing at that branch, worktree, and
  artifact pair until the update is approved and merged

The guider authors `roadmap-update.md` and any roadmap bundle edits. The
reviewer checks that the update matches the merged round evidence, preserves
roadmap immutability, and keeps `state.json` activation metadata consistent.
The controller may activate a new roadmap revision only after the roadmap
update review approves it. After the roadmap-update branch merges and any
approved active revision metadata is applied, clear `state.json.roadmap_update`.

`roadmap-update.md` must record:

- source round id and merged commit
- roadmap id, prior revision, and proposed active revision
- files changed
- status changes or new revision summary
- evidence from the merged round that justifies the update
- whether `state.json` activation metadata must change

`roadmap-update-review.md` must record an explicit `APPROVED` or `REJECTED`
decision with evidence.

## Worktree Preparation

Prepare the repository for round worktrees during setup:

- `orchestrator/worktrees/` is the canonical location for round worktrees.
- Ensure `orchestrator/worktrees/` is ignored by a tracked ignore rule.
- Do not place machine state inside `orchestrator/worktrees/`.
- Keep the rest of `orchestrator/` in the main checkout so resume state, role files, and roadmap bundles stay visible.
