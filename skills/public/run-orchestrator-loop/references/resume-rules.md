# Resume Rules

Resume automatically from `orchestrator/state.json`.

If the repo exposes a retry contract file such as `orchestrator/retry-subloop.md`, read it before deciding how to resume review outcomes.

## Startup

1. Read `orchestrator/state.json`.
2. If `active_round_id` is null and `stage` is `done`, start a new round with the guider.
3. If a round is active, reopen the recorded branch and worktree and resume the recorded stage.
4. If repo-local machine state includes retry bookkeeping, resume the exact recorded attempt instead of guessing a new one.

## Retry Outcomes

- Keep the same `active_round_id`.
- Keep the same branch and worktree.
- Return from `review` to `plan` whenever the repo-local review contract requests retry.
- Treat accepted-but-retryable review outcomes the same as rejected retries for controller purposes: do not merge, do not advance the roadmap, and do not create a new round.
- Increment the attempt number only when the repo-local machine state or review contract says to do so.

## Interrupted Stages

- Resume the exact incomplete stage.
- Resume the exact recorded retry attempt when retry state is active.
- Use a fresh subagent for the resumed stage.
- Do not create a replacement round just because a stage was interrupted.

## Missing Worktree

- Recreate the worktree from the recorded branch when possible.
- If the branch or worktree metadata is unusable, record a recoverable `resume_error` in `state.json` and stop instead of guessing.

## Corrupt State

- Do not invent missing fields.
- Record the controller error.
- Ask the user for help only after you have captured the precise problem in state.
