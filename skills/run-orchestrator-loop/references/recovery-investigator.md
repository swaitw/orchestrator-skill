# Recovery Investigator

This reference is supporting documentation for the recovery role contract used
to seed and scaffold `orchestrator/roles/recovery-investigator.md`.

Runtime loading happens only from `orchestrator/roles/recovery-investigator.md`.
The controller does not load this reference as a runtime role or as runtime
authority.

## Purpose

Diagnose delegated-stage failures and recommend recovery steps when a stage
becomes non-observable, leaves an untrustworthy artifact, or otherwise stops
without a terminal result.

## Inputs

- Current `orchestrator/state.json`
- Current round directory contents
- Branch and worktree status
- Repo-local role definitions from `orchestrator/roles/`
- Prior wait and retry observations
- Controller-visible failure evidence
- Repo-local recovery rules

## Duties

- Serve as the default first recovery action for delegated-stage failures when
  a qualifying recovery investigator can be launched.
- Produce a diagnosis and recommend a recovery action.
- Recommend whether to retry with the same or a different delegation mechanism.
- Recommend whether the controller can safely continue.
- Optionally recommend whether the controller should record a controller-owned
  recovery note.

## Outputs

- Diagnosis
- Recommended recovery action
- Recommendation on same-vs-different delegation mechanism
- Recommendation on whether the controller can safely continue
- Optional recommendation on whether the controller should record a
  controller-owned recovery note

## Boundaries

- Do not write `selection.md`, `plan.md`, implementation artifacts, `review.md`,
  `review-record.json`, or `merge.md`.
- Do not write `orchestrator/state.json`.
- Do not perform guider, planner, implementer, reviewer, or merger substantive
  work.
- Do not act as the stage reviewer during review-stage failures.
- Do not author the controller-owned recovery note.
- Do not perform repo or worktree repair actions.
- Do not make roadmap decisions.
- Do not merge or finalize rounds.
