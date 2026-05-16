# ADR-0006: Deepen machine contracts for orchestration records

## Status

Accepted, partially superseded by ADR-0009 and ADR-0010

ADR-0009 removes the human merge note after round finalization becomes
controller-owned. ADR-0010 removes the prose selection handoff after selection
becomes planner-owned.

## Context

After ADR-0002 minimized top-level controller state and ADR-0005 made
status-only closeout controller-owned, several smaller shallow Interfaces
remained:

- a persisted merge-readiness flag duplicated reviewer approval, closeout
  validity, dependency ordering, base freshness, and semantic roadmap-update
  serialization.
- selected lineage and scheduling appeared in both prose and `state.json`.
- terminal detection and closeout anchors still depended on Markdown headings
  and insertion text.
- review and closeout schemas repeated lineage and edit-shape rules.
- recovery-investigator launch rules were split between a role prompt and a
  controller reference.

Each duplicated rule made the workflow harder to resume safely because the
controller had to reconcile fields that were supposed to describe the same
truth.

## Decision

Deepen the machine contracts:

- remove persisted merge readiness; derive merge admissibility at runtime;
- make `selection-record.json` the machine authority for selected lineage and
  scheduler fields;
- add `roadmap-view.json` as the machine view for terminal detection, roadmap
  ids, direction ids, dependencies, and closeout anchors;
- replace separate review/closeout schema docs with
  `round-finalization-schema.md`;
- collapse recovery-investigator controller-use rules into the repo-local
  recovery role contract.

`state.json` stays focused on live controller state. Human Markdown files stay
available for handoff and review, but runtime decisions cross the smaller
machine Interfaces.

## Consequences

**Positive:**
- Merge admissibility cannot drift from a stale boolean.
- Selected lineage and scheduler fields have one machine source.
- Roadmap terminal detection no longer depends on brittle Markdown parsing.
- Round finalization edit-shape changes happen in one schema document.
- Recovery launch and consumption rules have one repo-local contract.

**Negative:**
- Scaffolds gain `selection-record-schema.md`, `round-finalization-schema.md`,
  and active-bundle `roadmap-view.json`.
- Existing control planes need migration if they still rely on persisted merge
  readiness or the older split finalization schema files.

**Neutral:**
- `roadmap.md` and `review.md` remain human-facing artifacts. ADR-0009 and
  ADR-0010 later remove the extra prose handoff artifacts.
- `review-record.json` and `closeout-record.json` remain separate artifacts
  because their owners differ.
