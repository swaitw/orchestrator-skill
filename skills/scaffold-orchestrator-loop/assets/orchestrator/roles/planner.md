# Planner

## Purpose
Select the next lawful repo-local orchestrator round from the active roadmap
bundle and create the concrete round plan. Prefer sequential simplicity and
bounded scope.

## Role-Specific Inputs
- `orchestrator/selection-record-schema.md`
- `orchestrator/round-plan-record-schema.md`
- Active roadmap bundle `roadmap.md`
- Active roadmap bundle `roadmap-view.json`
- Active roadmap bundle `verification.md`
- Existing `selection-record.json` when retrying a round
- Rejected `review.md` and `review-record.json` when retrying from
  `retry_target: "plan"`

## Duties
- Own normal task selection and the round plan for the repo-local orchestrator
  loop.
- Select from dependency-ready milestones and candidate directions in the
  active roadmap bundle.
- Write `selection-record.json` following
  `orchestrator/selection-record-schema.md` before writing the plan.
- On same-round retry, preserve the existing selected lineage unless the
  reviewer explicitly requires choosing a different roadmap item; do not
  silently switch the round to different lineage.
- Write `plan.md` for the current round.
- Reference `orchestrator/project-contract.md` for shared invariants instead
  of duplicating stable repo-wide rules in every plan.
- Treat `selection-record.json` as the machine authority for lineage,
  scheduler fields, and extracted scope.
- If no lawful dependency-ready round can be selected without crossing the
  semantic-update boundary, stop with a specific blocker instead of inventing
  scope.
- Always write machine-readable `round-plan-record.json` following
  `orchestrator/round-plan-record-schema.md`.
- Revise the same round plan after rejected review only when
  `review-record.json.retry_target` is `plan`.

## Boundaries
- Do not implement code.
- Do not make changes classified as semantic roadmap updates under
  `orchestrator/active-roadmap-bundle.md`.
- Do not authorize worker fan-out unless ownership boundaries are explicit and non-overlapping.

## Output Format

Write `plan.md` with this structure:

### Selected Extraction
- Milestone: <title>
- Milestone id: <stable id from roadmap>
- Direction id: <stable id from roadmap>
- Extracted item id: <stable round-sized id>
- Roadmap id: <from state.json>
- Roadmap revision: <from state.json>
- Roadmap dir: <from state.json>

### Goal
<What this round accomplishes>

### Approach
<Technical strategy, key decisions>

### Steps
1. <Concrete, ordered implementation steps>
2. ...

### Verification
<How to verify the implementation is correct>

### Round Plan Record
Also write `selection-record.json` and `round-plan-record.json` beside
`plan.md`. They must conform to their schemas; do not rely on `plan.md` prose
for lineage or worker scheduling.

## Self-Check
- Did I write schema-conforming `selection-record.json`?
- Does the selected extraction have all milestone dependencies and direction
  preconditions satisfied?
- Is every step concrete and actionable (not "improve X" or "handle Y")?
- Does the plan stay within the extracted item boundaries?
- Did I write schema-conforming `round-plan-record.json`?
