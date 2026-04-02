# Roadmap As Strategy-Backlog Design

## Context

The current orchestrator contract treats `roadmap.md` too much like an ordered
task list. That makes the roadmap read like a plan, even though the system
already has separate guider, planner, and implementer roles.

This design changes the contract so the roadmap becomes a larger control-plane
document:

- the roadmap defines direction, coordination boundaries, and candidate work
- the guider extracts concrete round-sized work from that roadmap
- the planner turns extracted work into an executable plan
- the implementer executes that plan

The goal is to make the roadmap broader than a plan while keeping the runtime
traceable and safe for concurrent teams.

## Goals

- Make `roadmap.md` a strategic coordination document rather than a flat plan.
- Preserve a clear handoff chain from roadmap to guider to planner to
  implementer.
- Allow the guider to select from any dependency-ready milestone, not only the
  first open one.
- Allow the guider to refine candidate directions into concrete round items
  without rewriting the roadmap every time.
- Allow the guider to select multiple concurrent items when dependencies,
  ownership boundaries, and merge constraints make that safe.
- Keep roadmap revisions durable and meaningful when a refinement changes future
  coordination.

## Non-Goals

- Replace `plan.md` with roadmap content.
- Turn the roadmap into a fully specified task list for implementers.
- Remove existing review and merge controls.
- Require concurrency for every repository.

## Recommended Model

Use a single roadmap document that combines strategy and backlog source
material.

The roadmap should contain milestones that are larger than a round and should
not be directly implementable without guider refinement. Each milestone contains
candidate directions that indicate plausible work fronts, but those directions
remain lighter than an execution plan.

This creates the intended handoff chain:

`roadmap.md` -> guider extraction in `selection.md` -> planner `plan.md` ->
implementer execution

## Roadmap Responsibilities

`roadmap.md` owns:

- overall goal
- outcome boundaries
- global sequencing rules
- milestone ordering and dependency shape
- known parallel lanes
- candidate directions within milestones

`roadmap.md` does not own:

- step-by-step execution details
- worker fan-out specifics
- implementation sequencing inside a selected round

## Roadmap Structure

The active roadmap bundle should keep a single `roadmap.md`, but the document
shape should change from `## Items` to a layered format.

Top-level sections:

- `Goal`
- `Outcome boundaries`
- `Global sequencing rules`
- `Parallel lanes`
- `Milestones`

Each milestone should be a coordination unit with metadata and explanation:

- `Milestone id`
- `Title`
- `Status`
- `Depends on`
- `Intent`
- `Completion signal`
- `Parallel lane`
- `Coordination notes`
- `Candidate directions`

Each candidate direction should remain lighter than a plan but structured enough
for guider extraction:

- `Direction id`
- `Summary`
- `Why it matters now`
- `Preconditions`
- `Parallel hints`
- `Boundary notes`
- `Extraction notes`

## Guider Responsibilities

The guider should shift from "pick the next task" to "extract the next
executable slice or concurrent batch from the roadmap."

The guider may:

- select from any milestone whose dependencies are satisfied
- choose any candidate direction whose preconditions are satisfied
- refine a candidate direction into a concrete round item if the extracted slice
  stays within the direction's stated boundaries
- select several items at once when they are safe to run concurrently

The guider should use roadmap-declared parallel lanes when they exist, but may
also assemble additional concurrent sets when boundaries are explicit and merge
ordering remains lawful.

The guider must not invent work that falls outside the milestone intent or
direction boundaries.

## Planner Responsibilities

The planner remains the owner of `plan.md`.

The planner receives concrete extracted work from `selection.md` and turns it
into:

- a bounded round goal
- technical approach
- ordered implementation steps
- verification steps
- optional `worker-plan.json` only when worker fan-out is justified

The planner should not be responsible for discovering strategic directions from
the roadmap. That discovery belongs to the guider.

## Implementer Responsibilities

The implementer remains downstream of the planner.

The implementer should execute one extracted round item or one approved
concurrent slice assignment, not reinterpret the roadmap directly. This keeps
the roadmap at the coordination layer and preserves the planner as the owner of
execution detail.

## Selection Artifact Changes

`selection.md` should record extraction lineage, not only a selected item title.

Recommended fields:

- selected milestone id
- selected direction id
- extracted round item id
- extracted round item summary
- extracted boundaries
- concurrency batch context when relevant
- rationale for why this slice should run now
- active `roadmap_id`
- active `roadmap_revision`
- active `roadmap_dir`

This makes each round traceable to the roadmap source while still allowing the
guider to refine the work into something planner-ready.

## State Schema Direction

The machine state should continue to support legacy repositories, but the
conceptual model should move away from treating a roadmap entry as the direct
execution unit.

Preferred lineage fields for active rounds:

- `milestone_id`
- `direction_id`
- `extracted_item_id`

`roadmap_item_id` may remain temporarily as a compatibility bridge for older
flat roadmap revisions, but new strategy-backlog revisions should treat the
extracted round item as the runtime execution unit.

## Parallel Selection Rules

The control-plane rules should support several teams pushing related work
together without collapsing the roadmap into a single linear plan.

Parallel selection is allowed when:

- dependencies for each selected item are satisfied
- ownership boundaries are clear enough for independent execution
- merge ordering is explicit where needed
- selected items do not create ambiguous overlapping scope

The roadmap should advertise known parallel lanes, but guider judgment remains
part of the model. The roadmap provides direction and obvious coordination
signals; the guider decides what should run now.

## Roadmap Revision Rule

Guider refinement should use a hybrid persistence model.

- Write the immediate extraction into `selection.md` for the current round.
- Publish a new roadmap revision only when the refinement changes future
  coordination, sequencing, boundaries, or shared interpretation.

This keeps the roadmap stable enough to guide the loop without forcing a new
revision for every small round-sized extraction.

## Compatibility And Migration

The system should preserve support for older flat roadmap revisions while making
new scaffolds use the larger strategy-backlog structure.

Migration direction:

- new scaffolded roadmaps use milestones plus candidate directions
- old flat revisions remain readable through compatibility handling
- runtime and reviewer checks continue to require stable lineage back to the
  active roadmap bundle
- role prompts should be updated so guider, planner, and reviewer use the new
  semantics consistently

## Acceptance Criteria

This design is successful when:

- `roadmap.md` clearly reads as a control-plane direction document rather than a
  round-by-round plan
- a guider can extract lawful round work from any dependency-ready milestone
- a planner can write a concrete `plan.md` without reinterpreting roadmap
  strategy
- multiple concurrent selections are possible when roadmap lanes or explicit
  guider reasoning make them safe
- roadmap revisions remain durable and are created only when future
  coordination meaning changes

## Follow-On Implementation Targets

The contract change should be reflected in at least these areas:

- `skills/scaffold-orchestrator-loop/references/roadmap-generation.md`
- `skills/scaffold-orchestrator-loop/references/repo-contract.md`
- `skills/scaffold-orchestrator-loop/assets/orchestrator/roadmaps/TEMPLATE_ROADMAP_ID/rev-001/roadmap.md`
- `skills/scaffold-orchestrator-loop/assets/orchestrator/roles/guider.md`
- `skills/scaffold-orchestrator-loop/assets/orchestrator/roles/planner.md`
- runtime and verification docs that still assume a flat roadmap item model

