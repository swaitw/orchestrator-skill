# Repo Contract

Scaffold a visible top-level `orchestrator/` directory in the target repository. The required repo contract lives under `orchestrator/`, including role definitions and worktree preparation.

## Required Files

- `orchestrator/state.json`
- `orchestrator/roadmaps/<roadmap_id>/<roadmap_revision>/roadmap.md`
- `orchestrator/roadmaps/<roadmap_id>/<roadmap_revision>/retry-subloop.md`
- `orchestrator/roadmaps/<roadmap_id>/<roadmap_revision>/verification.md`
- `orchestrator/roles/guider.md`
- `orchestrator/roles/planner.md`
- `orchestrator/roles/implementer.md`
- `orchestrator/roles/reviewer.md`
- `orchestrator/roles/merger.md`
- `orchestrator/roles/recovery-investigator.md`
- `orchestrator/rounds/`
- `orchestrator/worktrees/`

## `state.json` Schema

Use a small machine-oriented state file that supports safe serial defaults plus
explicit parallel execution.

`roadmap_id` naming rule:

- `YYYY-MM-DD` is the roadmap family's creation date in the repo-local controller context.
- `NN` is the zero-based roadmap-family ordinal for that day: `00`, `01`, `02`, and so on.
- `<slug>` is the stable descriptive control-plane family slug.
- Runtime should preserve this value verbatim once scaffolded; do not recompute it from the roadmap title later.

| Key | Type | Meaning |
| --- | --- | --- |
| `base_branch` | string | Branch that accepted rounds merge into |
| `roadmap_id` | string | Stable identifier for the active roadmap family in `YYYY-MM-DD-NN-<slug>` form |
| `roadmap_revision` | string | Active roadmap revision such as `rev-001` |
| `roadmap_dir` | string | Active roadmap bundle directory under `orchestrator/roadmaps/` |
| `stage` | string or null | Backward-compatible mirror of the single active round stage |
| `controller_stage` | string | Controller stage such as `dispatch-rounds`, `update-roadmap`, `done`, or `blocked` |
| `max_parallel_rounds` | number | Hard cap on concurrent live rounds |
| `active_round_id` | string or null | Preferred next round to resume |
| `active_rounds` | array | Canonical live round records |
| `pending_merge_rounds` | array | Round ids currently paused at `pending-merge` |
| `current_task` | string or null | Legacy mirror of the single active round task summary |
| `branch` | string or null | Legacy mirror of the single active round branch name |
| `worktree_path` | string or null | Legacy mirror of the single active round worktree path |
| `active_round_dir` | string or null | Legacy mirror of the single active round directory |
| `round_artifacts` | object | Legacy mirror of the single active round artifact map |
| `last_completed_round` | string or null | Most recently accepted round |
| `resume_error` | string or null | Last recoverable controller error |
| `resume_errors` | object | Structured controller and per-round recoverable errors |

Each `active_rounds[]` record should carry the machine-owned fields needed to
resume a round safely:

| Key | Type | Meaning |
| --- | --- | --- |
| `round_id` | string | Stable round identifier |
| `roadmap_item_id` | string | Stable selected roadmap item id |
| `selected_roadmap_revision` | string | Revision the round was selected from |
| `selected_roadmap_dir` | string | Roadmap bundle path the round was selected from |
| `current_task` | string | Human-readable selected task summary |
| `stage` | string | `select-task`, `plan`, `implement`, `review`, `pending-merge`, `merge`, `done`, or `blocked` |
| `branch` | string | Canonical round branch |
| `worktree_path` | string | Canonical round worktree |
| `active_round_dir` | string | Canonical round artifact directory |
| `round_artifacts` | object | Canonical round artifact path map |
| `depends_on_round_ids` | array | Round ids that must merge first |
| `merge_after_item_ids` | array | Item ids that must merge first |
| `parallel_group` | string or null | Co-scheduling group for the round |
| `worker_mode` | string | `none`, `fanout`, or `integrate` |
| `worker_records` | object | Worker branch, worktree, artifact, and status map |
| `merge_ready` | boolean | Whether the round may enter `merge` now |
| `resume_error` | string or null | Per-round recoverable error |

Legacy compatibility rules:

- Old serial repositories remain valid.
- If parallel fields are missing, runtime should normalize them to safe serial
  defaults instead of treating them as fatal corruption.
- A legacy roadmap revision without explicit `Item id:` fields may derive
  revision-local internal ids such as `legacy-item-<ordered-number>` until a
  later roadmap revision is authored with explicit item ids.

## File Ownership

- The orchestrator may update `orchestrator/state.json` and round bookkeeping only.
- The scaffolded role files under `orchestrator/roles/` are required repo contract files. Tailor them during setup and keep them tracked in the main checkout.
- Delegated role agents author roadmap bundle content, round artifacts, review records, and merge notes.
- The guider owns task selection and roadmap updates.
- Once any round has used a roadmap revision, keep that revision immutable. Publish a new revision directory instead of rewriting a used revision.

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
`roadmap_dir`, and `roadmap_item_id` in `selection.md` and `review-record.json`
so archived packets remain self-contained.

When planner-authored worker fan-out is active, the round should also contain:

- `worker-plan.json`
- `workers/<worker-id>/assignment.md`
- `workers/<worker-id>/implementation-notes.md`
- `workers/<worker-id>/handoff.md`

`worker-plan.json` is controller-readable machine state for worker scheduling.
The `workers/` subtree is human-facing evidence owned by delegated workers.

## Worktree Preparation

Prepare the repository for round worktrees during setup:

- `orchestrator/worktrees/` is the canonical location for round worktrees.
- Ensure `orchestrator/worktrees/` is ignored by a tracked ignore rule.
- Do not place machine state inside `orchestrator/worktrees/`.
- Keep the rest of `orchestrator/` in the main checkout so resume state, role files, and roadmap bundles stay visible.
