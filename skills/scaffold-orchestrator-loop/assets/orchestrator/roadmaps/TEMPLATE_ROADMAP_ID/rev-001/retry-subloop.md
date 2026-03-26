# Retry Subloop Contract

Tailor this file for the active roadmap revision at
`orchestrator/roadmaps/TEMPLATE_ROADMAP_ID/rev-001/`.

If this repo does not use same-round retry yet, keep that rule explicit here
instead of deleting the file.

## Scope

- State whether same-round retry is allowed.
- If retry is allowed, name which roadmap items or stage outcomes may retry.

## Machine State

- Document any retry-specific fields required in `orchestrator/state.json`.

## Review Output

- State which review outcomes return the same round to `plan`.
- State which review outcomes finalize and allow `merge`.

## Roadmap Revision Rule

- If an accepted `update-roadmap` stage changes the roadmap contract
  semantically, author a new roadmap revision directory instead of rewriting a
  used revision.
