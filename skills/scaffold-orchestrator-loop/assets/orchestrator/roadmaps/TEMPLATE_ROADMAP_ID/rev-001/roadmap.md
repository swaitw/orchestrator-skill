# Roadmap

## Status Legend

- `pending`
- `in-progress`
- `done`

## Goal

Replace this example with the overall outcome for the roadmap family.

## Alignment Summary

- Approved thesis: Replace this example with the human-approved strategy this
  roadmap family will follow.
- Success criteria: Replace this example with the concrete signals that make
  the roadmap meaningfully complete.
- Non-goals: Replace this example with adjacent work the scaffold alignment
  intentionally excluded.
- Chosen strategy: Replace this example with the approved roadmap approach and
  why it was chosen over the alternatives.
- Deferred alternatives: Replace this example with options considered during
  alignment but not selected for this family.

## Outcome Boundaries

- In scope: Replace this example with the delivery fronts this roadmap owns.
- Out of scope: Replace this example with adjacent work that should stay outside
  this roadmap family.
- Repo-wide invariants: see `orchestrator/project-contract.md`; this roadmap
  should record only roadmap-specific overrides.
- Completed history: keep compact notes in
  `orchestrator/roadmaps/TEMPLATE_ROADMAP_ID/roadmap-history.md` instead of
  copying completed item bodies into new active revisions.

## Global Sequencing Rules

- Replace this example with cross-cutting ordering or merge constraints.

## Parallel Lanes

- `lane-core`: Replace this example with the first obvious concurrent delivery
  lane.

## Milestones

### [pending] Milestone 1: Replace this example with the first coordination milestone
Milestone id: milestone-001-example
Depends on:
Intent: Establish the first coherent delivery front for this roadmap family.
Completion signal: The first delivery front is complete and no longer needs new
extracted rounds.
Parallel lane: lane-core
Coordination notes: Keep extracted rounds small enough that multiple teams can
work without overlapping scope.

Candidate directions:
- Direction id: direction-001-example
  Summary: Extract the first bounded round from this milestone.
  Why it matters now: This is the most direct way to start the milestone.
  Preconditions:
  Parallel hints: Serial by default; may co-run only with explicitly lane-safe
    work.
  Boundary notes: Keep the extraction focused on one reviewable outcome.
  Extraction notes: Preserve milestone intent while narrowing to one
    round-sized slice.

- Direction id: direction-002-example
  Summary: Replace this example with a second candidate direction in the same
    milestone.
  Why it matters now: Use this when a different bounded slice should move
    before or alongside the first extraction.
  Preconditions:
  Parallel hints: May run with other lane-core extractions only when ownership
    boundaries are explicit.
  Boundary notes: Keep shared interfaces or merge ordering explicit in the
    extracted round.
  Extraction notes: Record why this direction was chosen over the other ready
    options.

### [pending] Milestone 2: Replace this example with a dependent follow-up milestone
Milestone id: milestone-002-example
Depends on: milestone-001-example
Intent: Build on the earlier delivery front without precommitting the exact
round sequence.
Completion signal: Follow-up capability is integrated and no further extraction
from this milestone is required.
Parallel lane: lane-expansion
Coordination notes: Do not start extraction until the first milestone exposes
the needed boundaries or shared interfaces.

Candidate directions:
- Direction id: direction-003-example
  Summary: Replace this example with the first follow-up direction.
  Why it matters now: This becomes ready after the preceding milestone creates
    the necessary base.
  Preconditions: milestone-001-example complete
  Parallel hints: Start serially unless the planner can show clear ownership
    separation from other ready work.
  Boundary notes: Keep this focused on the new delivery front rather than
    reopening earlier milestone scope.
  Extraction notes: If this crosses the semantic-update boundary in
    `orchestrator/active-roadmap-bundle.md`, publish a new roadmap revision
    instead of hiding the change only in `selection-record.json`.
