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

Use a small machine-oriented state file.

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
| `active_round_id` | string or null | Current round identifier |
| `stage` | string | `done`, `select-task`, `plan`, `implement`, `review`, `merge`, or `update-roadmap` |
| `current_task` | string or null | Selected roadmap item summary |
| `branch` | string or null | Active round branch name |
| `worktree_path` | string or null | Active round worktree path |
| `active_round_dir` | string or null | Folder under `orchestrator/rounds/` for the active round |
| `round_artifacts` | object | Artifact path map for the active round |
| `last_completed_round` | string or null | Most recently accepted round |
| `resume_error` | string or null | Last recoverable controller error |

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

Every round must record the active `roadmap_id`, `roadmap_revision`, and `roadmap_dir` in `selection.md` and `review-record.json` so archived packets remain self-contained.

## Worktree Preparation

Prepare the repository for round worktrees during setup:

- `orchestrator/worktrees/` is the canonical location for round worktrees.
- Ensure `orchestrator/worktrees/` is ignored by a tracked ignore rule.
- Do not place machine state inside `orchestrator/worktrees/`.
- Keep the rest of `orchestrator/` in the main checkout so resume state, role files, and roadmap bundles stay visible.
