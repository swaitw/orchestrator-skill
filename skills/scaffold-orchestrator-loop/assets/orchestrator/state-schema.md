# State Schema

This document describes the fields in `orchestrator/state.json`.

## Top-Level Fields

| Key | Type | Description |
|-----|------|-------------|
| `base_branch` | string | Branch that accepted rounds merge into |
| `roadmap_id` | string | Stable family identifier for the active roadmap, normally `YYYY-MM-DD-NN-<slug>` |
| `roadmap_revision` | string | Active roadmap revision (e.g. `rev-001`) |
| `roadmap_dir` | string | Path to the active roadmap bundle directory |
| `stage` | string or null | Legacy mirror of the single active round stage |
| `controller_stage` | string | Controller stage: `dispatch-rounds`, `update-roadmap`, `done`, or `blocked` |
| `max_parallel_rounds` | number | Hard cap on concurrent live rounds (default `1` for serial) |
| `active_round_id` | string or null | Convenience pointer to the preferred next round to resume |
| `active_rounds` | array | Canonical live round records (see below) |
| `pending_merge_rounds` | array | Round ids currently blocked at `pending-merge` |
| `current_task` | string or null | Legacy mirror of the single active round task |
| `branch` | string or null | Legacy mirror of the single active round branch |
| `worktree_path` | string or null | Legacy mirror of the single active round worktree |
| `active_round_dir` | string or null | Legacy mirror of the single active round directory |
| `round_artifacts` | object | Legacy mirror of the single active round artifact map |
| `last_completed_round` | string or null | Most recently merged round id |
| `resume_error` | string or null | Controller-level recoverable error for legacy readers |
| `resume_errors` | object | Structured controller and per-round recoverable errors |
| `retry` | object or null | Controller-owned retry subloop state; `null` when no retry is active |

## Round Record Fields (each entry in `active_rounds`)

| Key | Type | Description |
|-----|------|-------------|
| `round_id` | string | Stable round identifier |
| `milestone_id` | string | Selected milestone that supplied the extracted work |
| `direction_id` | string | Candidate direction chosen by the guider |
| `extracted_item_id` | string | Stable id for the concrete round item derived from the selected direction |
| `roadmap_item_id` | string or null | Compatibility mirror for legacy flat-roadmap revisions |
| `selected_roadmap_revision` | string | Revision the round was selected from |
| `selected_roadmap_dir` | string | Bundle path the round was selected from |
| `current_task` | string | Human-readable summary of the extracted round scope |
| `stage` | string | One of: `select-task`, `plan`, `implement`, `review`, `pending-merge`, `merge`, `done`, `blocked` |
| `branch` | string | Canonical round branch |
| `worktree_path` | string | Canonical round worktree |
| `active_round_dir` | string | Canonical round artifact directory |
| `round_artifacts` | object | Canonical round artifact path map |
| `depends_on_round_ids` | array | Round ids that must merge before this round may merge |
| `merge_after_item_ids` | array | Extracted item ids that must merge before this round may merge |
| `parallel_group` | string or null | Co-scheduling group for this round |
| `worker_mode` | string | `none`, `fanout`, or `integrate` |
| `worker_records` | object | Worker state keyed by worker id |
| `merge_ready` | boolean | Whether this round may legally enter `merge` now |
| `resume_error` | string or null | Per-round recoverable error |

## Legacy Compatibility

When `active_rounds` has exactly one entry, the top-level legacy fields
(`stage`, `active_round_id`, `current_task`, `branch`, `worktree_path`,
`active_round_dir`, `round_artifacts`) may mirror that round for backward
compatibility.

When `active_rounds` has more than one entry, legacy fields must be cleared
to avoid ambiguity.

When `active_rounds` is empty, all legacy round fields should be null or empty.

For strategy-backlog roadmap revisions, round lineage should come from
`milestone_id`, `direction_id`, and `extracted_item_id`. Use
`roadmap_item_id` only as a compatibility bridge for older flat roadmap
revisions or older runtime readers.
