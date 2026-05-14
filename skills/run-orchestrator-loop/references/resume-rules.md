# Resume Rules

Resume automatically from `orchestrator/state.json`.

Resolve the active roadmap bundle from `state.json` `roadmap_id`,
`roadmap_revision`, and `roadmap_dir` before reading any human-facing control
files.

Load `orchestrator/active-roadmap-bundle.md` before interpreting
`roadmap.md`, `roadmap-view.json`, or `verification.md`.

Load `orchestrator/artifact-manifest.md` before resolving artifact paths.

## Startup

1. Read `orchestrator/state.json`.
2. If `roadmap_id`, `roadmap_revision`, or `roadmap_dir` is missing, record the
   controller error in `state.json.resume_errors.controller` and stop instead
   of guessing.
3. Treat `roadmap_id` as the scaffolded family identifier, normally in
   `YYYY-MM-DD-NN-<slug>` form; preserve it verbatim and do not recompute it
   from roadmap titles or directory names.
4. Load `orchestrator/active-roadmap-bundle.md`. If it is missing, record a
   migration-needed controller error in `state.json.resume_errors.controller`
   and stop instead of falling back to scattered roadmap rules.
5. Load `orchestrator/artifact-manifest.md`.
6. Resolve the active roadmap bundle from `roadmap_dir`.
7. If `state.json.roadmap_update` is not `null`, resume `update-roadmap` from
   that record before making any terminal `done` decision.
8. If `controller_stage` is `blocked`, if any live round stage is `blocked`, if
   any controller `resume_errors` entry exists, or if any owned-record
   `resume_error` exists for a live round or roadmap update,
   resume into automatic recovery for the matching controller, round, or
   roadmap-update context instead of stopping at the prior blockage note.
9. If `active_rounds` is empty and `controller_stage` is `done`, inspect the
   active roadmap bundle under `orchestrator/active-roadmap-bundle.md` before
   stopping or replying.
10. If `active_rounds` is empty, `controller_stage` is `done`, and the active
   roadmap bundle has unfinished milestones,
   treat that as a stale non-terminal `done` state and resume at
   `dispatch-rounds`.
11. If `active_rounds` is empty, `controller_stage` is `done`,
   `state.json.roadmap_update` is `null`, no unresolved resume errors remain,
   and the active roadmap bundle has no unfinished milestones, the
   controller may stop.
12. If live rounds exist, reopen each recorded branch and worktree, resume the
   recorded round stage, and recover round lineage from
   `selection-record.json`.
13. Before deciding a round-owned stage artifact is missing, inspect the
    recorded canonical round worktree and treat that worktree as the primary
    observation surface for `selection.md`, `selection-record.json`, `plan.md`,
    `round-plan-record.json`, `implementation-notes.md`, `review.md`,
    `review-record.json`, and `merge.md`.
14. If repo-local machine state includes retry bookkeeping, resume the exact
    recorded attempt instead of guessing a new one.
15. If the host exposes prior subagent handles for the recorded role/stage,
    prefer the compatible prior subagent under
    `delegation-boundaries.md` before launching a new one.

## Round Artifact Path Resolution

- Follow `orchestrator/artifact-manifest.md` for canonical artifact keys and
  path resolution.
- Treat every `round_artifacts` path in `state.json` as repo-relative unless it
  is already absolute.
- While a round is live, resolve repo-relative artifact paths against that
  round record's `worktree_path`, not the parent checkout.
- While a round is live, resolve repo-relative `active_round_dir` the same way:
  against that round record's `worktree_path`.
- The parent checkout copy is archival after merge. Its absence while a round is
  live is not evidence that a delegated stage failed.
- After a successful merge, resolve archived artifacts from the parent checkout
  unless the state still records a live round for that `round_id`.

## Retry Outcomes

- Keep the same `round_id`.
- Keep the same branch and worktree for the round.
- Return from `review` to `plan`, `implement`, or `review` exactly as the
  repo-local review contract requests.
- When a retry returns to `plan`, `implement`, or `review`, prefer the prior
  compatible planner, implementer, worker/integration implementer, or reviewer
  for that same round/branch/worktree. Use a fresh role subagent only when no
  compatible prior subagent is resumable or safely usable.
- Treat accepted-but-not-mergeable review outcomes as `pending-merge` for
  controller purposes: do not merge, do not advance the roadmap, and do not
  create a new round.
- Increment the attempt number only when the repo-local machine state or review
  contract says to do so.

## Pending-Merge Rounds

Use [state-machine.md](state-machine.md) as the authoritative transition table.
This section only defines the observations needed to choose the lawful
transition.

- If a round is in `pending-merge`, load `selection-record.json` from the
  canonical round worktree and re-check scheduler fields, dependency rounds,
  and base freshness before advancing. Interpret
  `scheduler.merge_after_item_ids` as extracted round ordering for
  strategy-backlog revisions.
- If a pending-merge round has `roadmap_closeout.mode == "status-only"`,
  revalidate `closeout-record.json` against the latest base branch and active
  roadmap bundle before allowing `merge`.
- If that revalidation fails, return the round to `closeout`, `review`, or
  recovery according to whether the approved closeout can still be applied.
- If a pending-merge round has
  `roadmap_closeout.mode == "semantic-update-required"` and
  `state.json.roadmap_update` is not `null`, keep it in `pending-merge` until
  the active semantic update is cleared.
- If drift requires substantive code refresh, return the round to `implement`.
- The whole-round implementer owns refresh when `worker_mode` is `none`.
- The integration implementer owns refresh when `worker_mode` is `fanout` or
  `integrate`.

## Worker Fan-Out

- If `worker_mode` is `fanout` or `integrate`, resume the exact incomplete
  worker or integration phase derived from `round-plan-record.json`, worker
  branch/worktree state, and worker artifacts.
- Use `round-plan-record.json` as the controller-readable source of truth for
  worker assignments, dependencies, branch/worktree paths, and integration
  ownership.
- Validate `round-plan-record.json` against
  `orchestrator/round-plan-record-schema.md` before launching or resuming
  workers.
- Do not infer worker ownership or dependency order from `plan.md` prose during
  resume.

## Roadmap Terminal Detection

- Follow `orchestrator/active-roadmap-bundle.md` for milestone status parsing,
  unfinished-work detection, and roadmap-view validation behavior. On
  validation error, record the exact controller error in
  `state.json.resume_errors.controller` instead of treating the roadmap as
  terminal.

## Status-Only Round Closeout Resume

Use [state-machine.md](state-machine.md) for legal transitions out of
`closeout`. This section only defines how to observe and repair closeout
evidence.

- If a live round stage is `closeout`, reopen the canonical round worktree and
  read its approved `review-record.json` plus the active `roadmap-view.json`.
- If `review-record.json` is missing, malformed, or lacks
  `roadmap_closeout.mode`, resume the round to `review` or recovery; do not
  merge and do not silently enter `update-roadmap`.
- If `roadmap_closeout.mode` is `status-only`, verify whether the canonical
  round worktree already contains the approved roadmap-view status updates,
  completion pointers, history entries, and matching `closeout-record.json`.
- If those exact status-only edits are missing, apply only the approved edits
  from `review-record.json` inside the canonical round worktree by resolving
  selectors through `roadmap-view.json`, then re-read the active roadmap bundle
  in that worktree, write `closeout-record.json`, and only then move the round
  to `pending-merge` or `merge`.
- If `roadmap_closeout.mode` is `semantic-update-required`, leave `closeout`
  and resume the round at `pending-merge` or `merge`; semantic roadmap changes
  happen after the round squash merge.
- If a previously merged round is discovered whose approved status-only
  closeout is not present on the base branch, record a precise controller
  recovery error in `state.json.resume_errors.controller` instead of applying
  post-merge edits directly.

## Roadmap Update Resume

- If `controller_stage` is `update-roadmap`, read `state.json.roadmap_update`
  under `orchestrator/roadmap-update-schema.md` and reopen the recorded
  roadmap-update branch/worktree when present, or recreate them from the base
  branch after the merged round when lawful.
- Observe `orchestrator/roadmap-updates/<round-id>-roadmap-update.md` and the
  matching review artifact before deciding whether the update is complete.
- Do not activate a new `roadmap_revision` or treat the roadmap update as done
  until `roadmap-update-review.md` explicitly approves it.
- If the update artifact exists but review is missing, dispatch the reviewer
  instead of mutating state directly.
- If `roadmap-update-review.md` rejects the update, keep the same
  `roadmap_update` branch and worktree, increment `roadmap_update.attempt`, set
  status to `rejected`, record `last_rejection_artifact` and
  `last_rejection_summary`, and re-dispatch the guider to revise
  `roadmap-update.md` and the proposed revision in place unless the review
  records a non-recoverable blockage.
- If the same roadmap update reaches 3 rejected attempts, set status to
  `blocked`, record the precise blocker in `roadmap_update.resume_error`, and
  do not dispatch another same-mechanism retry until recovery chooses a
  different lawful action.
- After the guider revises a rejected roadmap update, set status back to
  `review` and dispatch the reviewer again.
- After approved roadmap-update merge and state activation, clear
  `state.json.roadmap_update`.

## Non-Observable Stages

- If the active stage becomes non-observable or loses trustworthy
  controller-visible evidence, resume into recovery instead of recording
  immediate blockage.
- While recovering, the controller keeps the same round, same branch, same
  worktree, and same stage until controller-visible evidence or repo-local
  state lawfully changes them.
- The controller also preserves the retry attempt unless the missing stage or
  repo-local retry state is the thing that legitimately changes it.
- For any non-terminal delegated-stage stop, the controller must attempt a
  qualifying repo-local `recovery-investigator` from
  `orchestrator/roles/recovery-investigator.md` before treating the stop as
  terminal, unless it can record a deterministic reason why no available
  delegation mechanism can launch one at all.
- A live delegated agent that becomes non-observable is a recovery trigger, not
  permission to wait forever. After three consecutive non-interrupting
  observations with no status change, no artifact progress, and no trustworthy
  liveness signal, enter this recovery ladder.
- Recovery follows this ladder before any stop decision:
  1. re-read `state.json`, the active roadmap bundle, and the recorded branch /
     worktree;
  2. re-observe expected stage artifacts from the recorded canonical round
     worktree;
  3. if those artifacts already exist, clear stale blockage bookkeeping and
     resume from the stage those artifacts lawfully prove;
  4. if outputs are partial or stale, launch `recovery-investigator` and apply
     any controller-owned repair that makes already-produced outputs
     observable without authoring new stage content;
  5. re-dispatch the same stage on the same round / branch / worktree with
     explicit controller-visible output expectations, preferring the prior
     compatible role subagent when it is resumable or idle;
  6. if the prior compatible subagent remains non-observable or unusable,
     re-dispatch via a fresh role subagent or another lawful delegation
     mechanism;
  7. only step back a stage when recovered evidence or the repo-local retry
     contract lawfully requires it.
- Direct blockage is allowed only when the full recovery ladder yields no
  lawful next action, or when no qualifying `recovery-investigator` can launch
  through any available delegation mechanism, and the controller must record
  the precise blockage in `state.json.resume_errors.controller` before
  informing the user.
- A missing stage artifact by itself is not enough to justify stopping.
- Leave recovery only after controller-visible evidence shows whether the
  controller can safely resume, retry, or record a precise blockage.
- On a later controller pass, any previously recorded blockage must re-enter
  this recovery ladder automatically instead of stopping at the stale blockage
  note.

## Missing Worktree

- Recreate the worktree from the recorded branch when possible.
- If recreation succeeds, clear stale blockage bookkeeping and continue
  recovery from the same round/stage.
- If the branch, worktree, or active roadmap bundle metadata is unusable after
  lawful repair attempts, record a recoverable controller blockage in
  `state.json.resume_errors.controller` and stop instead of guessing.

## Corrupt State

- Do not invent missing fields.
- Record the controller error in `state.json.resume_errors.controller`.
- Ask the user for help only after you have captured the precise problem in
  state.
