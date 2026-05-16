---
name: run-orchestrator-loop
description: Use when `orchestrator/` and `orchestrator/state.json` exist in a repository and the delegated orchestrator loop needs to start or resume.
---

# Run Orchestrator Loop

## Overview

Act as a pure controller over the persisted repo-local orchestrator contract.
Start from [basic-serial-workflow.md](references/basic-serial-workflow.md),
load the repo-local control-plane contracts, delegate substantive work to role
subagents from `orchestrator/roles/`, and write only controller-owned state or
controller-owned artifacts.

The basic serial workflow is the front door. Load the advanced references only
when that file names an exit condition: recovery, worker fan-out, semantic
roadmap update, parallel rounds, or corrupt state.

This file is an entrypoint, not the complete runtime contract. When this file
and a referenced contract appear to overlap, the referenced contract owns the
detailed rule.

## Workflow

1. Load [basic-serial-workflow.md](references/basic-serial-workflow.md).
2. Follow its startup load and default workflow until it exits to an advanced
   Module.
3. At every delegated stage, load the shared role contract plus the role prompt
   named by the active stage.
4. Re-read `state.json` and the active roadmap bundle after every controller
   action before deciding whether to continue, exit, or stop.

## Reference Ownership

- [basic-serial-workflow.md](references/basic-serial-workflow.md) owns the
  common one-round-at-a-time path and the advanced-exit predicates.
- [state-machine.md](references/state-machine.md) owns controller stages, round
  stages, legal transitions, and stage ownership.
- [resume-rules.md](references/resume-rules.md) owns startup recovery,
  controller blockage handling, retry behavior, live worktree observation,
  worker fan-out resume, roadmap-update resume, and corrupt-state handling.
- [delegation-boundaries.md](references/delegation-boundaries.md) owns what the
  controller may do directly, what it must delegate, compatible subagent reuse,
  and subagent launch/observation rules.
- [worktree-finalization-rules.md](references/worktree-finalization-rules.md)
  owns merge admissibility, base freshness, squash-merge bookkeeping, and
  finalization checks.
- `orchestrator/active-roadmap-bundle.md` owns active bundle resolution,
  `roadmap-view.json` validation, terminal detection, status-only closeout
  limits, and semantic-revision rules.
- `orchestrator/artifact-manifest.md` owns file layout, artifact keys, and path
  resolution.
- `orchestrator/role-contract.md` owns shared role inputs, ownership, output
  rules, boundaries, and self-checks.
- Schema files under `orchestrator/` own their corresponding machine artifact
  shapes.

## Delegation Rule

Do not simulate delegated roles in the controller. For `plan`, `implement`,
`review`, `update-roadmap`, worker slices, integration implementation, and
recovery investigation, load the repo-local role prompt and delegate through
real subagents under [delegation-boundaries.md](references/delegation-boundaries.md).

## Completion

Continue until the active roadmap bundle is terminal under
`orchestrator/active-roadmap-bundle.md`, `state.json.active_rounds` is empty,
no active `state.json.roadmap_update` remains, and no unresolved resume errors
remain. Otherwise continue through the owning reference above, stop only for
explicit user interruption, or stop after the owning recovery rules record a
precise controller blockage.

## Pre-Completion Self-Check

Before sending a final response, verify ALL of these:
- If claiming terminal success: `state.json` `active_rounds` is empty
- If claiming terminal success: active roadmap bundle has no unfinished
  milestones under `orchestrator/active-roadmap-bundle.md`
- If claiming terminal success: no unresolved `resume_errors` entries or
  owned-record `resume_error` entries remain
- If stopping due to blockage: precise error recorded in
  `state.json.resume_errors.controller`
- If stopping due to user interruption: state is consistent and saved

## Resources

- [basic-serial-workflow.md](references/basic-serial-workflow.md)
- [state-machine.md](references/state-machine.md)
- [resume-rules.md](references/resume-rules.md)
- [delegation-boundaries.md](references/delegation-boundaries.md)
- [worktree-finalization-rules.md](references/worktree-finalization-rules.md)
