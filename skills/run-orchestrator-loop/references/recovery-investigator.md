# Recovery Investigator Controller Use

Runtime role loading happens only from
`orchestrator/roles/recovery-investigator.md`. This reference is for the
controller: when to launch that role, what context to provide, and how to use
its answer.

## Launch Conditions

Launch a qualifying recovery investigator when a delegated stage is
non-terminal and any of these are true:

- expected stage artifacts are missing, partial, stale, or untrustworthy;
- a subagent stopped without a controller-visible terminal result;
- persisted `blocked`, `resume_error`, or `resume_errors` state needs fresh
  recovery evaluation;
- the controller cannot tell whether existing worktree outputs already prove a
  lawful next stage.

Skip this launch only after recording a deterministic reason that no available
delegation mechanism can launch a qualifying recovery investigator.

## Context To Provide

Give the investigator the minimum evidence needed to diagnose the active
round/stage:

- current `orchestrator/state.json`;
- active roadmap bundle identity and retry policy;
- current round id, stage, branch, worktree, and round artifact paths;
- expected artifacts for the stage;
- observed artifact contents or absence;
- branch/worktree status;
- prior wait, retry, and blockage observations.

## Requested Output

Ask for:

- diagnosis grounded in observable evidence;
- whether existing stage outputs are salvageable and which stage they prove;
- recommended recovery action;
- whether to retry through the same or a different delegation mechanism;
- whether the controller can safely continue;
- optional controller-owned recovery note text.

## Consumption Rules

The controller may use the recommendation to clear stale blockage bookkeeping,
refresh artifact-path bookkeeping, recreate missing worktrees, or re-dispatch a
stage. The investigator does not make those state changes itself.

The investigator recommendation is advisory. The controller still applies the
state machine, retry policy, and delegation boundaries before changing state,
stepping a round backward, or recording blockage.

## Boundaries

The recovery investigator must not be asked to author delegated stage artifacts,
write `orchestrator/state.json`, perform implementation work, approve review
results, update the roadmap, merge, or finalize rounds.
