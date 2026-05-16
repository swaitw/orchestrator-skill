# ADR-0009: Collapse round finalization

## Status

Accepted

## Context

After adding the basic serial workflow, the default path still exposed too many
post-review stages and a dedicated delegated merge-preparation role.

Those concepts mostly described controller-owned bookkeeping. The actual
authority already lives in machine records:

- `review-record.json` for approval and closeout classification;
- `selection-record.json` for merge ordering;
- `round-plan-record.json` for worker fan-out;
- `closeout-record.json` for status-only closeout proof; and
- `state.json.roadmap_update` for semantic roadmap update serialization.

The dedicated merger role was therefore a shallow Module: deleting it does not
remove safety, because the same predicates are derivable from existing records.

## Decision

Replace the separate post-review stages with one controller-owned round stage:
`finalize-round`.

`finalize-round`:

- applies reviewer-approved status-only closeout when needed;
- writes compact `closeout-record.json`;
- derives merge admissibility from machine records and base freshness;
- performs squash merge bookkeeping;
- removes the live round from `state.json.active_rounds`; and
- dispatches semantic `update-roadmap` when the approved review requires it.

Remove the dedicated merge-preparation role and its human note artifact.

Also keep `state.json.active_rounds[]` smaller by deriving round summary and
lineage from `selection-record.json`, deriving round artifact paths from
`round_id`, `worktree_path`, and `orchestrator/artifact-manifest.md`, and by
deriving worker mode from `round-plan-record.json`.

## Consequences

**Positive:**
- The common workflow has one post-review Interface.
- Merge safety stays grounded in machine records instead of a human merge note.
- `state.json` has fewer duplicate fields.

**Negative:**
- Existing control planes with the older post-review stages or dedicated
  merge-preparation artifacts need migration.

**Neutral:**
- Reviewer approval remains delegated.
- Semantic roadmap updates remain delegated and reviewable.
