# State Schema

This document is the canonical field reference for `orchestrator/state.json`.
Other control-plane documents may point here, but should not duplicate the full
schema table.

Use a small machine-oriented state file that supports safe serial defaults plus
explicit parallel execution.

`roadmap_id` naming rule:

- `YYYY-MM-DD` is the roadmap family's creation date in the repo-local controller context.
- `NN` is the zero-based roadmap-family ordinal for that day: `00`, `01`, `02`, and so on.
- `<slug>` is the stable descriptive control-plane family slug.
- Runtime should preserve this value verbatim once scaffolded; do not recompute it from the roadmap title later.

## Top-Level Fields

| Key | Type | Description |
|-----|------|-------------|
| `contract_version` | string | Control-plane contract version. New scaffolds use `orchestrator-v2` |
| `base_branch` | string | Branch that accepted rounds merge into |
| `roadmap_id` | string | Stable family identifier for the active roadmap, normally `YYYY-MM-DD-NN-<slug>` |
| `roadmap_revision` | string | Active roadmap revision (e.g. `rev-001`) |
| `roadmap_dir` | string | Path to the active roadmap bundle directory |
| `controller_stage` | string | Controller stage: `dispatch-rounds`, `update-roadmap`, `done`, or `blocked`; new scaffolds with unfinished roadmap work start at `dispatch-rounds` |
| `max_parallel_rounds` | number | Hard cap on concurrent live rounds (default `1` for serial) |
| `active_rounds` | array | Canonical live round records (see below) |
| `roadmap_update` | object or null | Active semantic `update-roadmap` record following `orchestrator/roadmap-update-schema.md`, or `null` when no semantic roadmap update is in progress |
| `resume_errors` | object | Structured controller recoverable errors, including `controller` for controller-level blockage |
| `retry` | object or null | Controller-owned retry subloop state; `null` when no retry is active |

## Round Record Fields (each entry in `active_rounds`)

| Key | Type | Description |
|-----|------|-------------|
| `round_id` | string | Stable round identifier |
| `current_task` | string | Human-readable summary of the extracted round scope |
| `stage` | string | One of: `select-task`, `plan`, `implement`, `review`, `closeout`, `pending-merge`, `merge`, `done`, `blocked` |
| `branch` | string | Canonical round branch |
| `worktree_path` | string | Canonical round worktree |
| `active_round_dir` | string | Repo-relative canonical round artifact directory, resolved against `worktree_path` while the round is live and against the parent checkout only after merge |
| `round_artifacts` | object | Canonical round artifact path map using keys from `orchestrator/artifact-manifest.md`; repo-relative paths are resolved against `worktree_path` while live |
| `worker_mode` | string | `none`, `fanout`, or `integrate`; fan-out assignments and worker artifact paths are recorded in `round-plan-record.json` |
| `resume_error` | string or null | Per-round recoverable error |

## Invariants

When no semantic roadmap update is in progress, `roadmap_update` should be
`null`.
Only one semantic roadmap update may be active. If `roadmap_update` is not
`null`, additional rounds requiring semantic roadmap updates must wait in
`pending-merge`.
The record shape, branch/worktree convention, artifact paths, rejection loop,
and activation rules are defined by `orchestrator/roadmap-update-schema.md`.
Pending-merge queues are derived from live round records whose `stage` is
`pending-merge`.

Round lineage and merge-order scheduling come from the guider-authored
`selection-record.json` following `orchestrator/selection-record-schema.md`.
They are not duplicated in `state.json`.

Merge readiness is not persisted. The controller derives merge admissibility
from reviewer approval, closeout validity, scheduler fields in
`selection-record.json`, dependency state, base freshness, and active semantic
roadmap-update state.
