# ADR-0002: Minimize controller state shape

## Status

Accepted, amended by ADR-0007

## Context

The orchestrator control plane persists runtime coordination in
`orchestrator/state.json`. After ADR-0001 removed legacy-flat roadmap support and
top-level legacy mirror fields, the state shape still contained smaller
convenience fields that duplicated information already available in canonical
records:

- `active_round_id` duplicated resume preference that can be derived from
  `active_rounds[]`.
- `pending_merge_rounds` duplicated the set of live rounds whose own `stage` is
  `pending-merge`.
- top-level `resume_error` duplicated the structured `resume_errors` object.

These fields made the interface larger than the authority it represented. Any
disagreement between the convenience field and canonical records forced the
controller to decide which value to trust.

## Decision

Remove derived top-level controller state:

- Delete `active_round_id`; live rounds are tracked only in `active_rounds[]`.
  Resume preference is derived from the live round records.
- Delete `pending_merge_rounds`; pending-merge rounds are live rounds whose own
  `stage` is `pending-merge`.
- Delete top-level `resume_error`; controller-level blockage is recorded in
  `resume_errors.controller`.

Keep round-owned and roadmap-update-owned recovery fields in their owning
records:

- `active_rounds[].resume_error` remains for per-round recovery.
- `roadmap_update.resume_error` remains for active roadmap-update recovery.

## Consequences

**Positive:**
- `state.json` has fewer fields and fewer consistency rules.
- Round state remains local to each round record.
- Pending-merge and resume-selection logic become deterministic derived views.
- Controller-level blockage has one structured location.

**Negative:**
- Consumers that want a preferred active round or pending-merge queue must derive
  those views at read time.
- Existing control planes with the removed fields need migration before running
  the updated runtime.

**Neutral:**
- This ADR did not change roadmap lineage fields, round artifact paths,
  worker fan-out state, or roadmap-update state. ADR-0007 later moved those
  remaining shallow surfaces into deeper repo-local contracts.
- This ADR originally kept `last_completed_round` as compact summary metadata.
  ADR-0007 later removed it because merged round artifacts and roadmap history
  are the durable authorities.
