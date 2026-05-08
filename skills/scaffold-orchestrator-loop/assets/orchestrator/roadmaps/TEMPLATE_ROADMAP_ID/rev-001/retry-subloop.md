# Retry Subloop Policy

Active bundle: `orchestrator/roadmaps/TEMPLATE_ROADMAP_ID/rev-001/`. Tailor
this policy during setup.

This file records only repo- and roadmap-specific retry policy. Keep shared
runtime mechanics in the runtime skill references and controller state schema.

## Same-Round Retry

- Default: disabled.
- Enabled extracted items:
  - none
- Enabled review outcomes:
  - none
- Worker-slice retry:
  - disabled unless `worker-plan.json` explicitly enables it for the current
    round.

## Pending-Merge Policy

- A reviewed round may pause in `pending-merge` only for declared dependency
  ordering, extracted-item ordering, or base-branch freshness.
- If a `pending-merge` round needs substantive code refresh, return it to the
  owner named by the current round state: whole-round implementer when
  `worker_mode` is `none`, integration implementer when worker fan-out is
  active.

## Roadmap Revision Rule

- If `update-roadmap` changes future coordination, sequencing, boundaries, or
  retry meaning, publish a new roadmap revision directory instead of rewriting a
  used revision.

## Roadmap Overrides

- none
