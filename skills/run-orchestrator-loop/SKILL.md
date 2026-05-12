---
name: run-orchestrator-loop
description: Use when `orchestrator/` and `orchestrator/state.json` exist in a repository and the delegated orchestrator loop needs to start or resume.
---

# Run Orchestrator Loop

## Overview

Act as a pure controller over the persisted repo-local orchestrator contract:
read `orchestrator/state.json`, resolve the active roadmap bundle from
`roadmap_id`, `roadmap_revision`, and `roadmap_dir`, load runtime roles from
`orchestrator/roles/`, delegate substantive work to role subagents, reuse
compatible prior handles when available, and update only machine-control state.

Serial execution remains valid through `max_parallel_rounds: 1`, but runtime
requires the current `orchestrator-v2` strategy-backlog state shape. Missing
required state fields are migration-needed corrupt state; record a controller
error instead of synthesizing fields.
Parallel rounds and worker fan-out are allowed only when repo-local artifacts
explicitly authorize them.

Blocked state is recovery input, not permission to stop. Follow
[resume-rules.md](references/resume-rules.md) and
[delegation-boundaries.md](references/delegation-boundaries.md) for the
recovery ladder, blockage rules, and role boundaries.

## Workflow

1. Load state and references.
2. Apply [resume-rules.md](references/resume-rules.md), including recovery
   for persisted `blocked` / `resume_errors` state.
3. Use one `orchestrator/round-<nn>-<slug>` branch and one
   `orchestrator/worktrees/<round-id>` worktree per round.
4. Resume live rounds from `active_rounds[]` or dispatch new rounds up to
   `max_parallel_rounds`.
5. If a round plan authors `worker-plan.json`, launch worker implementers and
   an integration implementer exactly as required.
6. Move reviewed rounds through `pending-merge`, `merge`, and
   `update-roadmap` only when merge-order and base-branch rules allow it.
7. Re-read `state.json` plus the active roadmap bundle before deciding whether
   another round must start immediately.

## Load at Startup

- `orchestrator/state.json`
- `orchestrator/project-contract.md` when present
- Active roadmap bundle `roadmap.md` resolved from `state.json.roadmap_dir`
- Active roadmap bundle `retry-subloop.md` resolved from `state.json.roadmap_dir`
- [state-machine.md](references/state-machine.md)
- [resume-rules.md](references/resume-rules.md)

## Load Per Stage

- `select-task` / `update-roadmap`: `orchestrator/roles/guider.md`
- `update-roadmap` review: `orchestrator/roles/reviewer.md`
- `plan`: `orchestrator/roles/planner.md`
- `implement`: `orchestrator/roles/implementer.md`
- `review`: `orchestrator/roles/reviewer.md`, active roadmap bundle
  `verification.md`
- `merge`: `orchestrator/roles/merger.md`
- Recovery: `orchestrator/roles/recovery-investigator.md` and
  [recovery-investigator.md](references/recovery-investigator.md)
- Worker fan-out: `orchestrator/worker-plan-schema.md`
- Worktree/merge operations:
  [worktree-merge-rules.md](references/worktree-merge-rules.md) and
  [delegation-boundaries.md](references/delegation-boundaries.md)

## Stage Rules

Round stages:

- `select-task` belongs to the guider.
- `plan` belongs to the planner.
- `implement` belongs to the implementer.
- `review` belongs to the reviewer.
- `merge` uses merger-authored notes plus controller bookkeeping.

Controller-global stages:

- `dispatch-rounds` schedules or resumes live rounds.
- `update-roadmap` authoring belongs to the guider and must produce
  `orchestrator/roadmap-updates/<round-id>-roadmap-update.md`; approval belongs
  to the reviewer and must produce the matching `roadmap-update-review.md`.
- `done` is terminal only when the active roadmap parser finds no unfinished
  work and there are no live rounds.

Do not simulate these roles in your own voice.

## Controller Rules

- Update only machine-control state directly.
- Require `roadmap_id`, `roadmap_revision`, and `roadmap_dir` in
  `orchestrator/state.json`; if any are missing or unusable, stop and record
  the exact controller error in `state.json.resume_errors.controller` instead
  of guessing.
- Treat `roadmap_id` as an opaque scaffolded identifier, usually
  `YYYY-MM-DD-NN-<slug>`; preserve it verbatim and never recompute it from
  titles, paths, or directory names.
- Resolve live roadmap files only from `state.json.roadmap_dir`.
- Resolve live round artifact paths according to
  [resume-rules.md](references/resume-rules.md): repo-relative artifact paths
  are read from the recorded round `worktree_path` while the round is live.
- After `update-roadmap`, re-read `orchestrator/state.json`, resolve the active
  roadmap bundle again from `roadmap_dir`, and immediately start the next round
  when unfinished milestones remain and the concurrency cap allows it.
- Treat controller `done` as terminal only when the roadmap has no unfinished
  milestones, there are no live rounds, or user interruption lawfully stops
  progress.
- If the guider authored a new roadmap revision during `update-roadmap`,
  activate it by updating `state.json` `roadmap_id`, `roadmap_revision`, and
  `roadmap_dir` only after the roadmap update reviewer approves it.
- See [delegation-boundaries.md](references/delegation-boundaries.md) and
  [state-machine.md](references/state-machine.md) for complete rules.

## Recovery Rules

Use [resume-rules.md](references/resume-rules.md) as the complete recovery
procedure. In short: re-observe the recorded round worktree, preserve the same
round/stage while lawful, launch the repo-local `recovery-investigator` unless
deterministically impossible, and record blockage only after no lawful recovery
action remains.

## Subagent Rules

Follow [delegation-boundaries.md](references/delegation-boundaries.md) for
subagent launch, waiting, and role-ownership rules. Do not simulate delegated
roles in the controller.

## Completion

Continue until every roadmap milestone in the active bundle is complete or a
controller error recorded in `state.json.resume_errors.controller` lawfully
blocks safe progress. Do not stop just because one round reaches `done`;
terminal completion requires a roadmap re-check confirming no unfinished
milestones and no live rounds, or explicit user interruption. A recorded
blockage by itself is not terminal; it must follow the recovery and stop rules
in [resume-rules.md](references/resume-rules.md).

## Common Mistakes

- Simulating roles by writing `selection.md`, `plan.md`, or `review.md` yourself.
- Skipping `recovery-investigator` and recording blockage after one delegation failure.
- Treating `blocked` or recoverable error state as a terminal stop instead of an
  automatic recovery entry.
- Inventing parallelism by launching rounds without roadmap metadata authorization.
- Guessing missing state by inferring `roadmap_id` from directory names.
- Treating `pending-merge` as done and proceeding before merge-order is satisfied.
- Authoring delegated artifacts by writing `implementation-notes.md` or `merge.md` yourself.
- Mutating roadmap status or activating a new revision without delegated
  `roadmap-update.md` and approved `roadmap-update-review.md`.

## Pre-Completion Self-Check

Before sending a final response, verify ALL of these:
- If claiming terminal success: `state.json` `active_rounds` is empty
- If claiming terminal success: active roadmap bundle has no unfinished milestones
- If claiming terminal success: no unresolved `resume_errors` entries or
  owned-record `resume_error` entries remain
- If stopping due to blockage: precise error recorded in
  `state.json.resume_errors.controller`
- If stopping due to user interruption: state is consistent and saved

## Operational Limits

- Maximum 3 consecutive same-mechanism retry attempts per round before
  escalating to a different lawful recovery action; do not stop on
  same-mechanism exhaustion alone
- Maximum 50 tool calls per delegated stage before pausing for assessment
- If a single round has cycled through review -> plan -> implement -> review
  more than 3 times, record the pattern and ask the user only after the
  recovery ladder cannot narrow the problem further

## Resources

- [state-machine.md](references/state-machine.md)
- [resume-rules.md](references/resume-rules.md)
- [recovery-investigator.md](references/recovery-investigator.md)
- [delegation-boundaries.md](references/delegation-boundaries.md)
- [worktree-merge-rules.md](references/worktree-merge-rules.md)
