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
- Delegated role agents author roadmap bundle content, round selection and
  planning artifacts, implementation artifacts, and review records.
- The controller may apply only reviewer-approved status-only round closeout to
  the active roadmap revision.
- The planner owns normal task selection and round planning. The guider owns
  semantic roadmap updates.
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

Each round folder should contain round artifacts only. Use
`orchestrator/artifact-manifest.md` for the canonical round artifact set,
artifact keys, and live-vs-archived path resolution. Add extra files only when a
round needs them.

Every round must record the selection lineage fields from
`orchestrator/selection-record-schema.md` in `selection-record.json` and
`review-record.json` so archived packets remain self-contained.

`selection-record.json` must follow `orchestrator/selection-record-schema.md`.
`round-plan-record.json` must follow
`orchestrator/round-plan-record-schema.md`.
`review-record.json` must follow `orchestrator/round-finalization-schema.md`.
Status-only closeout rounds must also include controller-authored
`closeout-record.json` following `orchestrator/round-finalization-schema.md`.

When planner-authored worker fan-out is active, follow
`orchestrator/round-plan-record-schema.md` for worker metadata and integration
lifecycle. The `workers/` subtree is human-facing evidence owned by delegated
workers.

## Roadmap Closeout And Update Artifacts

During `finalize-round`, closeout classification follows
`orchestrator/active-roadmap-bundle.md`, and reviewer/controller records follow
`orchestrator/round-finalization-schema.md`. A round with missing or invalid
`roadmap_closeout` is not mergeable.

Semantic roadmap updates follow `orchestrator/roadmap-update-schema.md`.

## Worktree Preparation

Prepare the repository for round worktrees during setup:

- `orchestrator/worktrees/` is the canonical location for round worktrees.
- Ensure `orchestrator/worktrees/` is ignored by a tracked ignore rule.
- Do not place machine state inside `orchestrator/worktrees/`.
- Keep the rest of `orchestrator/` in the main checkout so resume state, role files, and roadmap bundles stay visible.
