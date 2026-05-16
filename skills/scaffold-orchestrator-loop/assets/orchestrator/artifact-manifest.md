# Artifact Manifest

This file owns orchestrator file layout, artifact keys, and path resolution.

## Shared Control-Plane Files

The scaffolded `orchestrator/` directory must contain:

- `state.json`
- `state-schema.md`
- `artifact-manifest.md`
- `project-contract.md`
- `active-roadmap-bundle.md`
- `role-contract.md`
- `selection-record-schema.md`
- `round-plan-record-schema.md`
- `round-finalization-schema.md`
- `roadmap-update-schema.md`
- `roles/guider.md`
- `roles/planner.md`
- `roles/implementer.md`
- `roles/reviewer.md`
- `roles/recovery-investigator.md`
- `rounds/`
- `roadmap-updates/`
- `roadmaps/`
- `worktrees/`

## Active Roadmap Bundle Files

The active roadmap revision directory named by `state.json.roadmap_dir` must
contain:

- `roadmap.md`
- `roadmap-view.json`
- `verification.md`

The roadmap family directory must contain:

- `roadmap-history.md`

## Round Artifact Keys

Round artifact paths are derived from `round_id`, not persisted in
`state.json`. Schema files may name the artifact path for their record format;
this manifest owns live-vs-archived path resolution. Use these keys:

| Key | Path |
|-----|------|
| `selection_record` | `orchestrator/rounds/<round-id>/selection-record.json` |
| `plan` | `orchestrator/rounds/<round-id>/plan.md` |
| `round_plan_record` | `orchestrator/rounds/<round-id>/round-plan-record.json` |
| `implementation_notes` | `orchestrator/rounds/<round-id>/implementation-notes.md` |
| `review` | `orchestrator/rounds/<round-id>/review.md` |
| `review_record` | `orchestrator/rounds/<round-id>/review-record.json` |
| `closeout_record` | `orchestrator/rounds/<round-id>/closeout-record.json` |

`closeout_record` is required only for status-only closeout rounds.

## Worker Artifact Paths

When `round-plan-record.json` authorizes worker fan-out, worker artifacts use:

- `orchestrator/rounds/<round-id>/workers/<worker-id>/assignment.md`
- `orchestrator/rounds/<round-id>/workers/<worker-id>/implementation-notes.md`
- `orchestrator/rounds/<round-id>/workers/<worker-id>/handoff.md`

Worker branch and worktree fields belong to
`orchestrator/round-plan-record-schema.md`; this manifest owns artifact paths
only.

## Roadmap Update Artifacts

Semantic roadmap-update branch, worktree, update artifact, and review artifact
paths are defined by `orchestrator/roadmap-update-schema.md`. Load that schema
instead of copying those conventions into callers.

## Path Resolution

- Treat manifest paths as repo-relative unless explicitly absolute.
- While a round is live, resolve round artifact paths inside the round record's
  `worktree_path`.
- After a successful round merge, resolve archived round artifacts from the
  parent checkout.
- Resolve active roadmap bundle files from `state.json.roadmap_dir`.
- Resolve roadmap-update artifacts inside the recorded roadmap-update worktree
  while an update is active.
