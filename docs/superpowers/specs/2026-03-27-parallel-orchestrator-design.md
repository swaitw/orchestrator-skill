# Parallel Orchestrator Skill Design

## Goal

Extend the orchestrator skill set so it can run:

- multiple independent roadmap items in parallel as separate rounds
- multiple explicit worker slices in parallel inside a single round stage

The design must keep concurrency explicit in repo-local state and artifacts
rather than inferred from chat context or controller heuristics.

## Current Problems

- The runtime contract is single-round and single-active-subagent.
- `orchestrator/state.json` models only one active round at a time.
- The roadmap format cannot explicitly declare safe parallel work.
- The planner role is written as a strictly sequential round planner.
- The implementer role owns one undivided round, with no worker-slice contract.
- The reviewer and merger contracts assume one linear round lifecycle.
- The scaffold skill generates a serial-only contract for new repositories.

These constraints make the current skill set predictable, but they prevent safe
parallel execution even when independent work is obvious.

## Decision

Parallelism will be opt-in and controller-visible.

The controller must never infer that two roadmap items or two worker slices are
safe to run concurrently. Parallel execution is allowed only when repo-local
artifacts explicitly authorize it.

The base rules are:

- roadmap items remain serial unless the roadmap marks them parallel-safe
- multiple rounds may run concurrently only when dependency and grouping rules
  allow it
- worker fan-out inside a round is allowed only when the round plan explicitly
  defines safe slices, ownership, and recomposition
- review and merge remain conservative and repo-visible

## Compatibility Rule

The runtime skill should remain usable on already-scaffolded serial
repositories.

Compatibility requirements:

- missing parallel metadata in `roadmap.md` means serial execution
- missing additive parallel fields in `orchestrator/state.json` should be
  initialized by the controller to safe serial defaults instead of treated as
  fatal corruption
- existing repositories with one active round should continue to run unchanged
  unless repo-local artifacts are revised to opt into concurrency

The scaffold skill will generate the full new schema for freshly scaffolded
repositories.

## Roadmap Contract

The active roadmap bundle remains the scheduler's source of truth.

Each roadmap item keeps:

- ordered item number
- `Item id:`
- status marker
- `Depends on:`
- `Completion notes:`

Each item also gains explicit concurrency metadata:

- `Parallel safe:` `yes` or `no`
- `Parallel group:` stable identifier or `none`
- `Merge after:` comma-separated item ids or `none`

Default behavior:

- unmarked items are treated as `Parallel safe: no`
- items with unfinished dependencies cannot start
- only items in the same non-`none` `Parallel group:` may run together
- merge order is derived from `Depends on:` plus explicit `Merge after:`
  entries

This keeps concurrency decisions in the human-reviewed roadmap file instead of
controller inference.

## State Model

`orchestrator/state.json` must grow from a single-active-round model into a
bounded multi-round scheduler state.

Top-level schema:

| Key | Type | Meaning |
| --- | --- | --- |
| `base_branch` | string | Branch that accepted rounds merge into |
| `roadmap_id` | string | Scheduler-active roadmap family identifier |
| `roadmap_revision` | string | Scheduler-active roadmap revision |
| `roadmap_dir` | string | Scheduler-active roadmap bundle path |
| `stage` | string or null | Backward-compatible mirror of the single active round stage |
| `controller_stage` | string | Controller stage: `dispatch-rounds`, `update-roadmap`, `done`, or `blocked` |
| `max_parallel_rounds` | number | Hard cap on concurrent live rounds |
| `active_round_id` | string or null | Convenience pointer to the preferred next round to resume |
| `active_rounds` | array | Canonical live round records |
| `pending_merge_rounds` | array | Round ids currently blocked at `pending-merge` |
| `last_completed_round` | string or null | Most recently merged round |
| `resume_error` | string or null | Controller-level recoverable error for legacy readers |
| `resume_errors` | object | Structured controller and per-round recoverable errors |
| `current_task` | string or null | Legacy mirror of the single active round task |
| `branch` | string or null | Legacy mirror of the single active round branch |
| `worktree_path` | string or null | Legacy mirror of the single active round worktree |
| `active_round_dir` | string or null | Legacy mirror of the single active round directory |
| `round_artifacts` | object | Legacy mirror of the single active round artifact map |

Each `active_rounds[]` record must include:

| Key | Type | Meaning |
| --- | --- | --- |
| `round_id` | string | Stable round identifier |
| `roadmap_item_id` | string | Stable selected roadmap item id |
| `selected_roadmap_revision` | string | Revision the round was selected from |
| `selected_roadmap_dir` | string | Bundle path the round was selected from |
| `current_task` | string | Human-readable selected task summary |
| `stage` | string | `select-task`, `plan`, `implement`, `review`, `pending-merge`, `merge`, `done`, or `blocked` |
| `branch` | string | Canonical round branch |
| `worktree_path` | string | Canonical round worktree |
| `active_round_dir` | string | Canonical round artifact directory |
| `round_artifacts` | object | Canonical round artifact path map |
| `depends_on_round_ids` | array | Round ids that must merge before this round may merge |
| `merge_after_item_ids` | array | Item ids that must merge before this round may merge |
| `parallel_group` | string or null | Co-scheduling group for this round |
| `worker_mode` | string | `none`, `fanout`, or `integrate` |
| `worker_records` | object | Worker branch, worktree, artifact, and status map keyed by worker id |
| `merge_ready` | boolean | Whether this round may legally enter `merge` now |
| `resume_error` | string or null | Per-round recoverable error |

State invariants:

- `active_rounds` is the canonical live-round source.
- `pending_merge_rounds` is exactly the set of `round_id` values whose round
  record stage is `pending-merge`.
- if `active_rounds` is empty, `active_round_id` is `null`, top-level `stage`
  is `null`, and all legacy round mirror fields are `null` or empty
- if `active_rounds` has exactly one entry, legacy mirror fields may mirror that
  round for compatibility
- if `active_rounds` has more than one entry, legacy mirror fields must be
  cleared to avoid ambiguity

Already-scaffolded serial repositories normalize into this model by:

- setting `max_parallel_rounds` to `1`
- setting `controller_stage` to:
  - `done` when legacy `active_round_id` is `null` and legacy `stage` is `done`
  - `dispatch-rounds` when legacy `active_round_id` is non-null
  - `blocked` when legacy `resume_error` is non-null and controller progress is
    unsafe
- creating `active_rounds` from the legacy single-round fields when
  `active_round_id` is non-null, with `active_rounds[0].stage` copied from the
  legacy top-level `stage`
- leaving all concurrency-specific keys at their safe serial defaults

## Roadmap Revisions and Item Identity

Parallel scheduling requires stable item identity across revisions.

Requirements:

- every roadmap item must declare a stable `Item id:`
- every round record and round artifact must pin `roadmap_item_id`,
  `selected_roadmap_revision`, and `selected_roadmap_dir`
- used roadmap revisions remain immutable
- a newer roadmap revision may become scheduler-active while older rounds remain
  live only if it preserves the `Item id:` values and merge-order contract for
  those live rounds
- if activating a new revision would invalidate a live round's selected item or
  merge constraints, the controller must defer activation until those rounds are
  finalized or explicitly replanned under a new round

Legacy roadmap normalization:

- a pre-parallel roadmap revision that lacks `Item id:` remains runnable in
  serial mode
- for such legacy revisions, the controller may derive a stable internal item
  id as `legacy-item-<ordered-number>` within that pinned immutable revision
- derived legacy item ids are valid only inside that specific pinned revision
  and must be persisted in normalized round state and round artifacts once
  selected
- any newly authored roadmap revision must replace derived legacy item ids with
  explicit `Item id:` fields

## Scheduling Model

The runtime scheduler remains deterministic.

Scheduling rules:

- launch parallel rounds only for roadmap items explicitly marked parallel-safe
- require all declared `Depends on:` entries to be `done` before scheduling an
  item
- enforce `max_parallel_rounds` as a hard cap
- keep each round on its own branch, worktree, and artifact directory
- allow different rounds to occupy different stages at the same time
- never skip the per-round stage order

Per-round stage order becomes:

1. `select-task`
2. `plan`
3. `implement`
4. `review`
5. `pending-merge`
6. `merge`
7. `done`

Controller-global stage order becomes:

1. `dispatch-rounds`
2. `update-roadmap`
3. `done`

The runtime may advance multiple rounds concurrently, but it may not invent
parallel stages inside a round unless the plan explicitly authorizes worker
fan-out.

## Transition and Resume Model

Per-round legal transitions:

- `select-task` -> `plan`
- `plan` -> `implement`
- `implement` -> `review`
- `review` -> `plan` when the repo-local review contract requests full-round
  retry
- `review` -> `pending-merge` when approval is granted but merge readiness is
  blocked by base freshness or declared merge ordering
- `review` -> `merge` when approval is granted and `merge_ready` is true
- `pending-merge` -> `merge` when merge blockers clear and the round is still
  review-valid
- `pending-merge` -> `implement` when base refresh or dependency drift requires
  substantive code refresh before merge
- `pending-merge` -> `review` when base refresh or dependency changes require
  renewed review evidence
- `pending-merge` -> `plan` when the repo-local retry contract requires a new
  plan after drift or worker retry
- `merge` -> `done`

Controller-global legal transitions:

- `done` -> `dispatch-rounds` when unfinished roadmap items remain
- `dispatch-rounds` -> `update-roadmap` after any successful round merge
- `update-roadmap` -> `dispatch-rounds` when unfinished items or live rounds
  remain
- `update-roadmap` -> `done` only when no unfinished items and no live rounds
  remain

Resume rules:

- normalize legacy serial state before scheduling
- reopen every live round's recorded branch and worktree
- resume the exact recorded round stage instead of guessing
- if a round is in `pending-merge`, re-check `merge_after_item_ids`, dependency
  rounds, and base freshness before advancing
- if a round leaves `pending-merge` for `implement`, the refresh owner is:
  - the whole-round implementer when `worker_mode` is `none`
  - the integration implementer when `worker_mode` is `fanout` or `integrate`
- if `worker_mode` is `fanout` or `integrate`, resume the exact incomplete
  worker or integration phase recorded in `worker_records`

## Parallel Round Model

Parallel branches are modeled as multiple live rounds.

For each live round, the controller must preserve:

- one round id
- one branch
- one worktree
- one round directory
- one canonical round `plan.md`
- one canonical round `review.md`
- one canonical round `merge.md`

Reviewer approval does not imply immediate merge. A round may become
`approved-pending-merge` when:

- another approved round must merge first because of explicit ordering
- the branch must be rebased or refreshed against the updated base branch
- a dependency round is not yet merged

The controller merges only rounds that are both approved and merge-ready under
the declared dependency graph.

## Parallel Stage Worker Model

Stage-worker fan-out is opt-in inside a round.

The planner may define a worker plan only when the round can be decomposed into
disjoint owned slices.

The plan must declare:

- worker identifiers
- each worker's exact owned files or module boundary
- required artifacts
- integration order
- any slice-specific checks
- the rule for recomposing worker outputs into the round branch

When `plan.md` declares worker slices, the controller may launch multiple
implementer workers in parallel for that round.

The controller must not invent worker slices on its own.

If the plan does not define safe slices, the round stays single-worker.

Whenever worker fan-out is used, the planner must also author a canonical
machine-readable `worker-plan.json`.

`worker-plan.json` is required controller input and must include:

- `round_id`
- `roadmap_item_id`
- `integration_owner`: `whole-round-implementer` or `integration-implementer`
- `workers[]`, where each worker entry declares:
  - `worker_id`
  - `owned_paths`
  - `required_artifacts`
  - `depends_on_workers`
  - `verification_commands`
  - `handoff_artifact`

Matching `worker_records` state must track, for each worker:

- `stage`: `queued`, `implementing`, `handoff-ready`, `integrating`, `done`, or
  `blocked`
- `branch`
- `worktree_path`
- `artifact_dir`
- `resume_error`

The controller launches and resumes workers from `worker-plan.json` and
`worker_records`, not by re-parsing free-form prose in `plan.md`.

Execution model:

- the round keeps one canonical round branch and one canonical round worktree
- each worker gets its own worker branch and worker worktree forked from the
  round branch baseline
- each worker implementer writes only inside its assigned worker branch/worktree
- after all worker implementers finish, the controller launches a dedicated
  integration implementer on the canonical round branch/worktree
- the integration implementer is responsible for composing worker results into
  the canonical round branch, resolving code-level conflicts, running any
  round-level checks required before review, and writing the canonical
  `implementation-notes.md`
- the controller may create worker branches and worktrees and record their
  state, but it must not perform substantive code integration itself

## Round Artifacts

The existing round-level artifacts remain canonical:

- `selection.md`
- `plan.md`
- `implementation-notes.md`
- `review.md`
- `review-record.json`
- `merge.md`

Parallel worker execution adds optional worker-scoped artifacts:

- `worker-plan.json`
- `workers/<worker-id>/assignment.md`
- `workers/<worker-id>/implementation-notes.md`
- `workers/<worker-id>/handoff.md`

The integrated round still ends with one canonical round-level review record
and one merge note. Worker artifacts are supporting evidence, not replacements
for the round contract.

## Role Contract Changes

### `guider`

The guider may nominate more than one next roadmap item only when the roadmap
explicitly marks them as parallel-safe and dependency-clean.

### `planner`

The planner remains responsible for the round plan, but may now author:

- a sequential single-worker plan
- a parallel worker plan with explicit ownership boundaries

The planner must not mark ambiguous work as parallel merely because it looks
independent.

### `implementer`

The implementer role must support two modes:

- whole-round implementation
- worker-slice implementation inside a planner-authored worker plan

Worker implementers own only their assigned slice and must not rewrite the
overall plan or claim round approval.

### `reviewer`

The reviewer must approve or reject the integrated round outcome, not just the
worker slices in isolation.

The canonical approval still happens at the round level after refresh,
integration, and round-level verification are complete.

### `merger`

The merger prepares only approved, merge-ready rounds. It must respect the
declared merge ordering and base-branch freshness requirements for parallel
rounds.

## Recovery and Retry

Parallelism must not weaken recovery rules.

`retry-subloop.md` should explicitly define:

- whether retry is allowed per round, per worker slice, or both
- whether a failed worker review requires full-round replanning or only
  slice-level retry
- how a reviewed-but-not-merged round re-enters execution after base-branch
  drift
- whether worker retry returns to `implement`, `review`, or `plan`

The controller should continue to treat incidental delegation failure as a
recovery problem, not an excuse to silently drop a round or worker slice.

## Skill Changes

### `scaffold-orchestrator-loop`

The scaffold skill will:

- scaffold the expanded `state.json` schema
- scaffold roadmap templates with parallel metadata fields
- scaffold role templates that describe round concurrency and worker-slice
  ownership
- scaffold retry-contract guidance covering both round retry and worker retry

### `run-orchestrator-loop`

The runtime skill will:

- normalize missing additive parallel fields to safe serial defaults
- load concurrency rules from the active roadmap bundle and round artifacts
- manage multiple live rounds concurrently within the configured cap
- launch parallel implementer workers only from explicit planner-authored worker
  plans
- keep review and merge decisions round-level and explicit

## README Changes

The README should describe the new model precisely:

- rounds may run in parallel only when the roadmap marks them parallel-safe
- stage-worker fan-out is opt-in and defined by round plans
- each round still uses one branch, one worktree, and one canonical review and
  merge record
- unmarked repositories continue to behave serially

## Verification

Verification should focus on contract integrity rather than executable code.

Required checks should include:

- `git diff --check`
- `rg` checks proving the serial-only language has been replaced where needed
- `rg` checks proving the new roadmap metadata and state keys are documented
- checks that scaffold assets, live skill docs, and README agree on the same
  concurrency model
- discovery verification that both skills still list correctly through the
  `skills` CLI

## Risks

- Parallel metadata that is underspecified will create ambiguous scheduler
  behavior.
- Worker-slice ownership that overlaps in practice will create merge churn and
  rework.
- A state schema that tries to preserve too much single-round compatibility can
  become confusing or internally inconsistent.
- If README, scaffold assets, and runtime skill drift, users will not know what
  the supported concurrency model actually is.

## Non-Goals

- implicit controller inference of safe parallel work
- unbounded fan-out across every pending roadmap item
- replacing round-level review with worker-level approvals
- hiding concurrency state in session memory instead of repo-local artifacts
