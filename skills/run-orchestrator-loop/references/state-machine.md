# State Machine

Use one strict linear round at a time.

The outer stage order stays linear even when a repo-local retry subloop is active inside one stage.

## Stages

1. `select-task`
2. `plan`
3. `implement`
4. `review`
5. `merge`
6. `update-roadmap`
7. `done`

## Ownership

- `select-task`: guider
- `plan`: planner
- `implement`: implementer
- `review`: reviewer
- `merge`: merger prepares notes, orchestrator performs bookkeeping
- `update-roadmap`: guider

## Legal Transitions

- `done` -> `select-task`
- `select-task` -> `plan`
- `plan` -> `implement`
- `implement` -> `review`
- `review` -> `plan` when the repo-local review contract requests retry
- `review` -> `merge` when the repo-local review contract approves finalization
- `merge` -> `update-roadmap`
- `update-roadmap` -> `select-task` when the active roadmap bundle still has unfinished `[pending]` or `[in-progress]` items
- `update-roadmap` -> `done` only when the active roadmap bundle has no unfinished items

If `update-roadmap` activates a new roadmap revision, the controller must update
`state.json` roadmap metadata before evaluating those transitions.

Do not skip forward and do not invent parallel stages.
