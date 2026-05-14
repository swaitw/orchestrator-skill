# ADR-0007: Collapse residual orchestrator contract surfaces

## Status

Accepted

## Context

After ADR-0006 deepened the round lineage, roadmap-view, finalization, and
recovery contracts, several residual shallow Interfaces still remained:

- artifact names and path-resolution rules were repeated across the repo
  contract, state schema, runtime rules, and role prompts;
- semantic roadmap-update branch, worktree, artifact, retry, and activation
  rules were split across state fields and role/runtime prose;
- `retry-subloop.md` was a required active-bundle file even when a roadmap had
  no roadmap-specific retry policy;
- each role prompt repeated the same input, ownership, output, and self-check
  structure;
- worker fan-out had both `worker-plan.json` and controller-owned
  `worker_records` state even though the planner's round plan already owned the
  worker assignment decision;
- `roadmap_style` remained in `state.json` even though ADR-0001 made
  Strategy-backlog the only supported roadmap shape; and
- `last_completed_round` remained as summary state even though merged round
  artifacts and roadmap history are the durable authorities.

The repeated surfaces made the control-plane Interface wider than the behavior
it exposed. Contract changes required synchronized edits across many callers
instead of one repo-local Module.

## Decision

Collapse the residual surfaces into deeper repo-local contracts:

- add `orchestrator/artifact-manifest.md` as the canonical file, artifact-key,
  and path-resolution contract;
- add `orchestrator/roadmap-update-schema.md` as the single semantic
  roadmap-update state and artifact contract;
- replace required `retry-subloop.md` with roadmap-specific retry policy under
  `verification.md` `## Roadmap Overrides`, while shared retry mechanics stay
  in runtime references;
- add `orchestrator/role-contract.md` for shared role input, ownership, output,
  and self-check rules;
- replace `worker-plan.json` with planner-authored
  `round-plan-record.json`, governed by
  `orchestrator/round-plan-record-schema.md`, so optional worker fan-out lives
  inside the round planning record;
- remove `roadmap_style` from `state.json`; `contract_version` and
  `orchestrator/active-roadmap-bundle.md` carry compatibility; and
- remove `last_completed_round` from `state.json`; merged round artifacts and
  roadmap history remain the authorities.

This amends ADR-0001's neutral choice to keep `roadmap_style` and ADR-0002's
neutral choice to keep `last_completed_round`.

## Consequences

**Positive:**
- Artifact paths and artifact ownership have one Interface.
- Semantic roadmap updates have one machine contract.
- Empty retry policy no longer creates a required active-bundle file.
- Role prompts become smaller and role-specific.
- Worker fan-out resume uses the round plan record and observable artifacts
  instead of duplicating worker state into `state.json`.
- `state.json` loses two derived or single-adapter fields.

**Negative:**
- Existing control planes with `roadmap_style`, `last_completed_round`,
  `worker-plan.json`, `worker_records`, or required `retry-subloop.md` need
  migration before running the updated contract.

**Neutral:**
- Strategy-backlog remains the only supported roadmap shape.
- Runtime still serializes semantic roadmap updates through
  `state.json.roadmap_update`; the shape now lives in
  `roadmap-update-schema.md`.
