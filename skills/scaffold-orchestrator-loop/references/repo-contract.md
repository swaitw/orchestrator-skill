# Repo Contract

Scaffold or advance a visible top-level `orchestrator/` directory in the target
repository. The required repo contract lives under `orchestrator/`, including
role definitions and worktree preparation.

## Setup Modes

`bootstrap` creates the initial repo-local control plane by copying the full
scaffold tree into a repository that does not yet have top-level
`orchestrator/`.

`next-family` reuses an existing terminal control plane. It must:

- detect `contract_version` before drafting the new family
- reuse existing `orchestrator/roles/`, `orchestrator/rounds/`, and
  `orchestrator/worktrees/`
- scaffold only missing shared control-plane files if the contract is
  incomplete
- create a fresh active bundle under
  `orchestrator/roadmaps/<roadmap_id>/rev-001/`
- preserve prior families, prior revisions, and prior round artifacts as
  immutable history

Do not use `next-family` when the existing control plane is still live or the
active roadmap bundle has unfinished milestones.

## Required Files

Use `orchestrator/artifact-manifest.md` as the canonical required-file,
artifact-key, and path-resolution contract. Bootstrap copies every shared file
listed there. `next-family` creates only the new active roadmap bundle and any
missing shared files listed there, except a missing
`orchestrator/active-roadmap-bundle.md`, which is migration-needed repair.

## `state.json` Schema

Use `orchestrator/state-schema.md` as the canonical field reference for
`orchestrator/state.json`. Do not restate the schema
table in roadmap bundle files or role prompts.

New scaffolds should write `contract_version: "orchestrator-v2"`.
Strategy-backlog is the only supported roadmap shape; do not persist a separate
roadmap-shape field.

`next-family` reset rules:

- preserve `base_branch`
- set the new `roadmap_id`, `roadmap_revision`, and `roadmap_dir`
- set `controller_stage` to `dispatch-rounds`
- clear `active_rounds`
- clear `roadmap_update`
- clear `resume_errors` and `retry`

## File Ownership

- The orchestrator may update `orchestrator/state.json` and round bookkeeping only.
- The scaffolded role files under `orchestrator/roles/` are required repo contract files. Tailor them during setup and keep them tracked in the main checkout.
- Approved scaffold alignment lives in the active roadmap bundle,
  `orchestrator/project-contract.md`, and `verification.md`. Do not leave
  strategy, success criteria, or non-goals only in chat history.
- Repo-wide invariants live in `orchestrator/project-contract.md`. Roadmaps and
  role files should point to that file instead of repeating shared event,
  fixture, dry-run, or package-boundary rules.
- Active roadmap bundle semantics live in
  `orchestrator/active-roadmap-bundle.md`. Runtime and role files should point
  there instead of repeating roadmap parsing, terminal detection, required
  bundle files, status-only closeout rules, or revision rules.
- File layout, artifact keys, and path-resolution rules live in
  `orchestrator/artifact-manifest.md`. Runtime and role files should point
  there instead of repeating artifact names.
- Shared role inputs, boundaries, output rules, and self-checks live in
  `orchestrator/role-contract.md`. Role files should contain only
  role-specific behavior and artifact formats.
- Semantic roadmap-update state, branch/worktree conventions, artifact formats,
  rejection handling, and activation rules live in
  `orchestrator/roadmap-update-schema.md`.
- Each `project-contract.md` section must contain concrete repo-specific
  entries or the exact phrase `none discovered yet`; blank headings are not a
  valid setup result.
- Delegated role agents author roadmap bundle content, round artifacts, review
  records, and merge notes.
- The controller may apply only reviewer-approved status-only round closeout to
  the active roadmap revision.
- The guider owns task selection and semantic roadmap updates.
- Once any round has used a roadmap revision, follow
  `orchestrator/active-roadmap-bundle.md` for status-only round closeout versus
  semantic new revision publication.
- Keep completed roadmap history in
  `orchestrator/roadmaps/<roadmap_id>/roadmap-history.md` or as compact
  completion pointers. Active revisions should not copy all completed item
  bodies forward.
- Do not widen a `next-family` setup into a full shared-contract refresh unless
  the user explicitly asks for that broader change.

## Round Artifacts

Each round folder should contain delegated artifacts only. Use
`orchestrator/artifact-manifest.md` for canonical artifact keys and path
resolution. The normal round artifact set is:

- `selection.md`
- `selection-record.json`
- `plan.md`
- `round-plan-record.json`
- `implementation-notes.md`
- `review.md`
- `review-record.json`
- `closeout-record.json` for status-only closeout rounds
- `merge.md`

Add more files only when a round needs them.

Every round must record the active `roadmap_id`, `roadmap_revision`,
`roadmap_dir`, `milestone_id`, `direction_id`, and `extracted_item_id` in
`selection-record.json` and `review-record.json` so archived packets remain
self-contained.

`selection-record.json` must follow `orchestrator/selection-record-schema.md`.
`round-plan-record.json` must follow
`orchestrator/round-plan-record-schema.md`.
`review-record.json` must follow `orchestrator/round-finalization-schema.md`.
Status-only closeout rounds must also include controller-authored
`closeout-record.json` following `orchestrator/round-finalization-schema.md`.

For live rounds, artifact paths recorded in `state.json` are repo-relative
paths resolved inside that round's `worktree_path`. The parent checkout sees
those artifacts only after the round branch is merged, so runtime must inspect
the recorded worktree first while a round is active.

When planner-authored worker fan-out is active, `round-plan-record.json`
records worker assignment, dependency, verification, branch, worktree, and
integration metadata. The `workers/` subtree is human-facing evidence owned by
delegated workers. The JSON shape, worker artifact paths, dependency fields,
and integration lifecycle are defined by
`orchestrator/round-plan-record-schema.md`.

## Roadmap Closeout And Update Artifacts

After a successful round merge, the controller reads the approved
`review-record.json`.

When `review-record.json` classifies the round as status-only, the controller
applies only the selected milestone status changes, compact completion
pointers, and compact history entries recorded there in the canonical round
worktree by resolving selectors through `roadmap-view.json` before squash
merge. The round branch carries those closeout edits into the squash merge. The
controller does not create a roadmap-update branch, does not set
`state.json.roadmap_update`, and does not change `roadmap_id`,
`roadmap_revision`, or `roadmap_dir`.

The controller must write `closeout-record.json` after applying status-only
closeout and must revalidate it before merge whenever the base branch or active
roadmap bundle changed while the round waited in `pending-merge`.

If an approving `review-record.json` lacks a valid `roadmap_closeout`, the
round is not mergeable. Return to review or recovery instead of treating the
missing field as a semantic roadmap update.

When `review-record.json` requires a semantic roadmap update, `update-roadmap`
is a delegated, reviewable stage. Use
`orchestrator/roadmap-update-schema.md` for the `state.json.roadmap_update`
shape, branch/worktree conventions, update artifact, review artifact,
rejection loop, and activation rules.

The guider authors `roadmap-update.md` and the proposed roadmap bundle revision.
The reviewer checks that the update matches the merged round evidence,
preserves roadmap immutability, and keeps `state.json` activation metadata
consistent. The controller may activate a new roadmap revision only after the
roadmap update review approves it. After the roadmap-update branch merges and
any approved active revision metadata is applied, clear
`state.json.roadmap_update`.

If the reviewer rejects `roadmap-update.md`, follow
`orchestrator/roadmap-update-schema.md`; do not start a new roadmap-update
branch unless recovery records the prior branch/worktree as unusable.

## Worktree Preparation

Prepare the repository for round worktrees during setup:

- `orchestrator/worktrees/` is the canonical location for round worktrees.
- Ensure `orchestrator/worktrees/` is ignored by a tracked ignore rule.
- Do not place machine state inside `orchestrator/worktrees/`.
- Keep the rest of `orchestrator/` in the main checkout so resume state, role files, and roadmap bundles stay visible.
