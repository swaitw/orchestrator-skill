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
     normalized state;
   - treat normalized `blocked`, `resume_error`, and `resume_errors` as
     recovery bookkeeping, not a terminal answer.
6. If `active_rounds` is empty and `controller_stage` is `done`, inspect the
   active roadmap bundle `roadmap.md` before stopping or replying.
7. If `active_rounds` is empty, `controller_stage` is `done`, and the active
   roadmap bundle still has unfinished `[pending]` or `[in-progress]`
   milestones,
   treat that as a stale non-terminal `done` state and resume at
   `dispatch-rounds`.
8. If `active_rounds` is empty, `controller_stage` is `done`, and the active
   roadmap bundle has no unfinished milestones, the controller may stop.
9. If `controller_stage` is `blocked`, if any live round stage is `blocked`, or
   if any `resume_error` / `resume_errors` entry exists for a live round,
   resume into automatic recovery on that same recorded round/stage instead of
   stopping at the prior blockage note.
10. If live rounds exist, reopen each recorded branch and worktree, resume the
   recorded round stage, and recover round lineage from `milestone_id`,
   `direction_id`, and `extracted_item_id`. Use `roadmap_item_id` only when
   the active roadmap revision is still a legacy flat roadmap.
11. Before deciding a round-owned stage artifact is missing, inspect the
    recorded canonical round worktree and treat that worktree as the primary
    observation surface for `selection.md`, `plan.md`, `implementation-notes.md`,
    `review.md`, `review-record.json`, and `merge.md`.
12. If repo-local machine state includes retry bookkeeping, resume the exact
    recorded attempt instead of guessing a new one.

## Round Artifact Path Resolution

- Treat every `round_artifacts` path in `state.json` as repo-relative unless it
  is already absolute.
- While a round is live, resolve repo-relative artifact paths against that
  round record's `worktree_path`, not the parent checkout.
- If the top-level legacy `worktree_path` mirror disagrees with the matching
  `active_rounds[]` record, prefer the round record.
- The parent checkout copy is archival after merge. Its absence while a round is
  live is not evidence that a delegated stage failed.
- After a successful merge, resolve archived artifacts from the parent checkout
  unless the state still records a live round for that `round_id`.

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
     explicit controller-visible output expectations;
  6. if the same mechanism remains non-observable, re-dispatch via a different
     lawful delegation mechanism;
  7. only step back a stage when recovered evidence or the repo-local retry
     contract lawfully requires it.
- Direct blockage is allowed only when the full recovery ladder yields no
  lawful next action, or when no qualifying `recovery-investigator` can launch
  through any available delegation mechanism, and the controller must record
  the precise blockage in `state.json` before informing the user.
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
  lawful repair attempts, record a recoverable `resume_error` in `state.json`
  and stop instead of guessing.

## Corrupt State

- Do not invent missing fields beyond the documented legacy normalization.
- Record the controller error.
- Ask the user for help only after you have captured the precise problem in
  state.
