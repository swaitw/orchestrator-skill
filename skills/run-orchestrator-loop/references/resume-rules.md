# Resume Rules

Resume automatically from `orchestrator/state.json`.

Resolve the active roadmap bundle from `state.json` `roadmap_id`,
`roadmap_revision`, and `roadmap_dir` before reading any human-facing control
files.

Read the active roadmap bundle `retry-subloop.md` before deciding how to
resume review outcomes.

## Startup

1. Read `orchestrator/state.json`.
2. If `roadmap_id`, `roadmap_revision`, or `roadmap_dir` is missing, record the
   controller error in `state.json` and stop instead of guessing.
3. Treat `roadmap_id` as the scaffolded family identifier, normally in
   `YYYY-MM-DD-NN-<slug>` form; preserve it verbatim and do not recompute it
   from roadmap titles or directory names.
4. Resolve the active roadmap bundle from `roadmap_dir`.
5. Normalize legacy serial state before scheduling:
   - if additive parallel fields are missing, create them with safe serial
     defaults;
   - if legacy `active_round_id` is non-null, create one `active_rounds[]`
     record from the legacy round fields;
   - set `controller_stage` to `done`, `dispatch-rounds`, or `blocked` from the
     normalized state.
6. If `active_rounds` is empty and `controller_stage` is `done`, inspect the
   active roadmap bundle `roadmap.md` before stopping or replying.
7. If `active_rounds` is empty, `controller_stage` is `done`, and the active
   roadmap bundle still has unfinished `[pending]` or `[in-progress]`
   milestones,
   treat that as a stale non-terminal `done` state and resume at
   `dispatch-rounds`.
8. If `active_rounds` is empty, `controller_stage` is `done`, and the active
   roadmap bundle has no unfinished milestones, the controller may stop.
9. If live rounds exist, reopen each recorded branch and worktree, resume the
   recorded round stage, and recover round lineage from `milestone_id`,
   `direction_id`, and `extracted_item_id`. Use `roadmap_item_id` only when
   the active roadmap revision is still a legacy flat roadmap.
10. If repo-local machine state includes retry bookkeeping, resume the exact
    recorded attempt instead of guessing a new one.

## Retry Outcomes

- Keep the same `round_id`.
- Keep the same branch and worktree for the round.
- Return from `review` to `plan`, `implement`, or `review` exactly as the
  repo-local review contract requests.
- Treat accepted-but-not-mergeable review outcomes as `pending-merge` for
  controller purposes: do not merge, do not advance the roadmap, and do not
  create a new round.
- Increment the attempt number only when the repo-local machine state or review
  contract says to do so.

## Pending-Merge Rounds

- If a round is in `pending-merge`, re-check `merge_after_item_ids`, dependency
  rounds, and base freshness before advancing. Interpret
  `merge_after_item_ids` as extracted round ordering for strategy-backlog
  revisions.
- If drift requires substantive code refresh, return the round to `implement`.
- The whole-round implementer owns refresh when `worker_mode` is `none`.
- The integration implementer owns refresh when `worker_mode` is `fanout` or
  `integrate`.

## Worker Fan-Out

- If `worker_mode` is `fanout` or `integrate`, resume the exact incomplete
  worker or integration phase recorded in `worker_records`.
- Use `worker-plan.json` plus `worker_records` as the controller-readable source
  of truth for worker scheduling and resume.
- Do not infer worker ownership or dependency order from `plan.md` prose during
  resume.

## Non-Observable Stages

- If the active stage becomes non-observable or loses trustworthy
  controller-visible evidence, resume into recovery instead of recording
  immediate blockage.
- While recovering, the controller keeps the same round, same branch, same
  worktree, and same stage until controller-visible evidence or repo-local
  state lawfully changes them.
- The controller also preserves `current_task` and the retry attempt unless the
  missing stage or repo-local retry state is the thing that legitimately
  changes them.
- For any non-terminal delegated-stage stop, the controller must attempt a
  qualifying repo-local `recovery-investigator` from
  `orchestrator/roles/recovery-investigator.md` before treating the stop as
  terminal, unless it can record a deterministic reason why no available
  delegation mechanism can launch one at all.
- Direct blockage is allowed only when the attempted `recovery-investigator`
  does not yield a qualifying recovery path, or when no qualifying
  `recovery-investigator` can launch through any available delegation
  mechanism, and the controller must record the precise blockage in `state.json`
  before informing the user.
- A missing stage artifact by itself is not enough to justify stopping.
- Leave recovery only after controller-visible evidence shows whether the
  controller can safely resume, retry, or record a precise blockage.

## Missing Worktree

- Recreate the worktree from the recorded branch when possible.
- If the branch, worktree, or active roadmap bundle metadata is unusable,
  record a recoverable `resume_error` in `state.json` and stop instead of
  guessing.

## Corrupt State

- Do not invent missing fields beyond the documented legacy normalization.
- Record the controller error.
- Ask the user for help only after you have captured the precise problem in
  state.
