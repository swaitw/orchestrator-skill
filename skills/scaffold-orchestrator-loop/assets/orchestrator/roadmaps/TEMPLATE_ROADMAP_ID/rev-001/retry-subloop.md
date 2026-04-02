# Retry Subloop Contract

Tailor this file for the active roadmap revision at
`orchestrator/roadmaps/TEMPLATE_ROADMAP_ID/rev-001/`.

If this repo does not use same-round retry yet, keep that rule explicit here
instead of deleting the file.

## Scope

- State whether same-round retry is allowed.
- If retry is allowed, name which extracted round items or stage outcomes may
  retry.
- State whether worker-slice retry is allowed when a plan uses `worker-plan.json`.
- State when a reviewed round may pause in `pending-merge` before merge.

## Machine State

- Document any retry-specific fields required in `orchestrator/state.json`.
- Document any worker record or `pending_merge_rounds` fields the controller
  must track.

## Review Output

- State which review outcomes return the same round to `plan`.
- State which review outcomes return the same round to `implement`.
- State which review outcomes return the same round to `review`.
- State which review outcomes finalize and allow `merge`.

## Roadmap Revision Rule

- If an accepted `update-roadmap` stage changes the roadmap contract
  semantically, author a new roadmap revision directory instead of rewriting a
  used revision.

## Pending-Merge Refresh

- State when base-branch drift or dependency merges send a round from
  `pending-merge` back to `implement`, `review`, or `plan`.
- State whether the whole-round implementer or the integration implementer owns
  the refresh when worker fan-out is active.
