# ADR-0005: Make status-only round closeout controller-owned

## Status

Accepted, partially superseded by ADR-0009

ADR-0009 keeps status-only closeout controller-owned but folds the separate
`closeout` stage into `finalize-round`.

## Context

The runtime loop used a delegated `update-roadmap` stage after every successful
round merge. That kept semantic roadmap changes reviewable, but it also made
pure completion bookkeeping pay the same branch, worktree, guider, and reviewer
cost as a real coordination change.

ADR-0004 already centralized the active roadmap bundle contract and
distinguished status-only evidence from changes that must publish a new
revision. The remaining friction was ownership: status-only evidence still
looked like a roadmap update even when the reviewer had already approved the
round and no future coordination meaning changed.

## Decision

Make status-only round closeout controller-owned.

After reviewer approval and before squash merge, the controller reads the
approved `review-record.json`. When that record explicitly classifies closeout
as `status-only`, the controller may apply only the exact milestone status
marker changes, compact completion pointers, and compact history entries
approved by the reviewer in the canonical round worktree. Those closeout edits
land in the same squash merge as the round.

The reviewer record and controller closeout record must follow
`orchestrator/round-finalization-schema.md`. Status-only selectors resolve
through the active bundle's `roadmap-view.json`. The controller records applied
closeout in `closeout-record.json` and revalidates that record before merge
after any base refresh.

The controller must not change future coordination, milestone or direction
meaning, sequencing, parallel lanes, extraction scope, verification meaning, or
retry policy during status-only closeout.

If the reviewer record is missing or ambiguous, the round is not mergeable and
runtime returns to review or recovery. If the reviewer record asks for a
semantic change, runtime skips status-only closeout and uses the delegated
`update-roadmap` path after the round merges. Semantic roadmap updates remain
guider-authored and reviewer-approved before activation.

## Consequences

**Positive:**
- Common post-merge completion bookkeeping no longer needs an extra branch,
  worktree, guider turn, and roadmap-update review.
- The reviewer is still the authority for whether a closeout is status-only.
- The controller has a durable `closeout-record.json` audit artifact.
- Semantic roadmap changes keep the stronger delegated and reviewable path.

**Negative:**
- `review-record.json` now has to carry a closeout classification.
- Status-only closeout adds a controller-owned evidence artifact.
- ADR-0009 later folds the deterministic closeout step into `finalize-round`.

**Neutral:**
- `state.json.roadmap_update` is still used for semantic updates.
- `roadmap_id`, `roadmap_revision`, and `roadmap_dir` do not change during
  status-only closeout.
