# ADR-0010: Collapse round preparation into planning

## Status

Accepted

## Context

After ADR-0009 collapsed post-review work into `finalize-round`, the remaining
pre-plan preparation stage became shallow. Its main responsibility was creating
`selection-record.json`, while the planner still had to read that record,
understand the selected scope, and author the actual plan.

Keeping selection in a separate stage made the normal workflow larger without
adding much leverage:

- the controller needed selection rules;
- the guider still had selection language;
- a prose selection handoff existed mostly as duplication; and
- the planner could not fully own the first substantive round decision.

## Decision

Remove the pre-plan preparation stage.

The controller still creates the round branch and worktree, but the first round
stage is now `plan`. The planner owns normal task selection and must write:

- `selection-record.json`;
- `plan.md`; and
- `round-plan-record.json`.

`selection-record.json` remains the machine authority for selected lineage and
merge-order scheduler fields. The extra prose handoff artifact is removed.

The guider no longer owns normal task selection. It remains responsible for
semantic `update-roadmap` when future coordination must change.

## Consequences

**Positive:**
- The default workflow starts with one substantive delegated role.
- Selection and planning have better Locality in the planner.
- The controller has fewer decision rules and no longer writes selected-round
  lineage.

**Negative:**
- Existing control planes with the older preparation stage or prose handoff
  artifact need migration.

**Neutral:**
- `selection-record.json` remains required for every round.
- The planner still may not change roadmap strategy; semantic changes remain
  delegated and reviewable through `update-roadmap`.
