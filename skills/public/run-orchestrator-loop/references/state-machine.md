# State Machine

Use one strict linear round at a time.

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
- `review` -> `plan` when rejected
- `review` -> `merge` when approved
- `merge` -> `update-roadmap`
- `update-roadmap` -> `done`

Do not skip forward and do not invent parallel stages.
