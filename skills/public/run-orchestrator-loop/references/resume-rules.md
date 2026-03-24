# Resume Rules

Resume automatically from `orchestrator/state.json`.

If the repo exposes a retry contract file such as `orchestrator/retry-subloop.md`, read it before deciding how to resume review outcomes.

## Startup

1. Read `orchestrator/state.json`.
2. If `active_round_id` is null and `stage` is `done`, inspect `orchestrator/roadmap.md` before stopping or replying.
3. If `active_round_id` is null, `stage` is `done`, and the roadmap still has unfinished `[pending]` or `[in-progress]` items, treat that as a stale non-terminal `done` state and start a new round with the guider.
4. If `active_round_id` is null, `stage` is `done`, and the roadmap has no unfinished items, the controller may stop.
5. If a round is active, reopen the recorded branch and worktree and resume the recorded stage.
6. If repo-local machine state includes retry bookkeeping, resume the exact recorded attempt instead of guessing a new one.

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
- Direct blockage is allowed only when no qualifying `recovery-investigator` can launch through any available delegation mechanism, and the controller must record the precise blockage in `state.json` before informing the user.
- Leave recovery only after controller-visible evidence shows whether the controller can safely resume, retry, or record a precise blockage.

## Interrupted Stages

- Resume the exact incomplete stage.
- Resume the exact recorded retry attempt when retry state is active.
- Use a fresh subagent for the resumed stage.
- Do not create a replacement round just because a stage was interrupted.
- If interruption leaves `stage: "done"` while the roadmap still has unfinished items, resume at `select-task` instead of treating the loop as complete.

## Missing Worktree

- Recreate the worktree from the recorded branch when possible.
- If the branch or worktree metadata is unusable, record a recoverable `resume_error` in `state.json` and stop instead of guessing.

## Corrupt State

- Do not invent missing fields.
- Record the controller error.
- Ask the user for help only after you have captured the precise problem in state.
