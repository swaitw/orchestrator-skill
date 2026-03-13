# Resume Rules

Resume automatically from `orchestrator/state.json`.

## Startup

1. Read `orchestrator/state.json`.
2. If `active_round_id` is null and `stage` is `done`, start a new round with the guider.
3. If a round is active, reopen the recorded branch and worktree and resume the recorded stage.

## Rejections

- Keep the same `active_round_id`.
- Keep the same branch and worktree.
- Return from `review` to `plan`.

## Interrupted Stages

- Resume the exact incomplete stage.
- Use a fresh subagent for the resumed stage.
- Do not create a replacement round just because a stage was interrupted.

## Missing Worktree

- Recreate the worktree from the recorded branch when possible.
- If the branch or worktree metadata is unusable, record a recoverable `resume_error` in `state.json` and stop instead of guessing.

## Corrupt State

- Do not invent missing fields.
- Record the controller error.
- Ask the user for help only after you have captured the precise problem in state.
