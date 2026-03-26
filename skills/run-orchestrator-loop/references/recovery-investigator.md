# Recovery Investigator

The recovery investigator is a shared-skill-owned real subagent used only for
diagnosis and recovery recommendations when a delegated stage becomes
non-observable, leaves an untrustworthy artifact, or otherwise creates a
non-terminal delegated-stage stop condition.

The controller should treat `recovery-investigator` as the default first
recovery action for delegated-stage failures. A controller may skip the launch
only when it records a deterministic reason why no available delegation
mechanism can launch a qualifying recovery investigator at all.

## Inputs

- current `orchestrator/state.json`
- current round directory contents
- branch/worktree status
- role definitions
- prior wait/retry observations
- controller-visible failure evidence
- shared recovery rules

## Outputs

- diagnosis
- recommended recovery action
- recommendation on same-vs-different delegation mechanism
- recommendation on whether the controller can safely continue
- optional recommendation on whether the controller should record a controller-owned recovery note

## Boundaries

- may not write `selection.md`, `plan.md`, implementation artifacts,
  `review.md`, `review-record.json`, or `merge.md`
- may not write `orchestrator/state.json`
- may not perform guider/planner/implementer/reviewer/merger substantive work
- may not act as the stage reviewer during review-stage failures
- may not author the controller-owned recovery note itself
- may not perform repo/worktree repair actions
- may not make roadmap decisions
- may not merge or finalize rounds
