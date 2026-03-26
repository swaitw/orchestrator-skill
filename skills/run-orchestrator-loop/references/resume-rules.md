# Resume Rules

Resume automatically from `orchestrator/state.json`.

Resolve the active roadmap bundle from `state.json` `roadmap_id`,
`roadmap_revision`, and `roadmap_dir` before reading any human-facing control
files.

Read the active roadmap bundle `retry-subloop.md` before deciding how to
resume review outcomes.

## Startup

1. Read `orchestrator/state.json`.
2. If `roadmap_id`, `roadmap_revision`, or `roadmap_dir` is missing, record the controller error in `state.json` and stop instead of guessing.
3. Treat `roadmap_id` as the scaffolded family identifier, normally in `YYYY-MM-DD-NN-<slug>` form; preserve it verbatim and do not recompute it from roadmap titles or directory names.
4. Resolve the active roadmap bundle from `roadmap_dir`.
5. If `active_round_id` is null and `stage` is `done`, inspect the active roadmap bundle `roadmap.md` before stopping or replying.
6. If `active_round_id` is null, `stage` is `done`, and the active roadmap bundle still has unfinished `[pending]` or `[in-progress]` items, treat that as a stale non-terminal `done` state and start a new round with the guider.
7. If `active_round_id` is null, `stage` is `done`, and the active roadmap bundle has no unfinished items, the controller may stop.
8. If a round is active, reopen the recorded branch and worktree and resume the recorded stage.
9. If repo-local machine state includes retry bookkeeping, resume the exact recorded attempt instead of guessing a new one.

## Retry Outcomes

- Keep the same `active_round_id`.
- Keep the same branch and worktree.
- Return from `review` to `plan` whenever the repo-local review contract requests retry.
- Treat accepted-but-retryable review outcomes the same as rejected retries for controller purposes: do not merge, do not advance the roadmap, and do not create a new round.
- Increment the attempt number only when the repo-local machine state or review contract says to do so.

## Non-Observable Stages

- If the active stage becomes non-observable or loses trustworthy controller-visible evidence, resume into recovery instead of recording immediate blockage.
- While recovering, the controller keeps the same round, same branch, same worktree, and same stage until controller-visible evidence or repo-local state lawfully changes them.
- The controller also preserves `current_task` and the retry attempt unless the missing stage or repo-local retry state is the thing that legitimately changes them.
- For any non-terminal delegated-stage stop, the controller must attempt a
  qualifying `recovery-investigator` before treating the stop as terminal,
  unless it can record a deterministic reason why no available delegation
  mechanism can launch one at all.
- Direct blockage is allowed only when the attempted
  `recovery-investigator` does not yield a qualifying recovery path, or when
  no qualifying `recovery-investigator` can launch through any available
  delegation mechanism, and the controller must record the precise blockage in
  `state.json` before informing the user.
- A missing stage artifact by itself is not enough to justify stopping.
- Leave recovery only after controller-visible evidence shows whether the controller can safely resume, retry, or record a precise blockage.

## Interrupted Stages

- Resume the exact incomplete stage.
- Resume the exact recorded retry attempt when retry state is active.
- Use a fresh subagent for the resumed stage.
- Do not create a replacement round just because a stage was interrupted.
- If interruption leaves `stage: "done"` while the active roadmap bundle still has unfinished items, resume at `select-task` instead of treating the loop as complete.

## Missing Worktree

- Recreate the worktree from the recorded branch when possible.
- If the branch, worktree, or active roadmap bundle metadata is unusable, record a recoverable `resume_error` in `state.json` and stop instead of guessing.

## Corrupt State

- Do not invent missing fields.
- Record the controller error.
- Ask the user for help only after you have captured the precise problem in state.
