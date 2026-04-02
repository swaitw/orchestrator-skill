---
name: run-orchestrator-loop
description: Use when `orchestrator/` and `orchestrator/state.json` exist in a repository and the delegated orchestrator loop needs to start or resume.
---

# Run Orchestrator Loop

## Overview

Act as a pure controller over the persisted repo-local orchestrator contract:
read `orchestrator/state.json`, resolve the active roadmap bundle from
`roadmap_id`, `roadmap_revision`, and `roadmap_dir`, load runtime roles from
`orchestrator/roles/`, delegate substantive work to fresh real subagents, and
update only machine-control state.

Serial repositories remain valid. If additive parallel fields are missing,
normalize to safe serial defaults instead of treating that as corruption.
Parallel rounds and worker fan-out are allowed only when repo-local artifacts
explicitly authorize them.

For incidental delegated-stage failure, attempt a qualifying
`recovery-investigator` before stopping. Record blockage in `state.json` only
after recovery is exhausted or deterministically unavailable.

## Workflow

1. Load state and references.
2. Normalize legacy serial state into safe controller defaults.
3. Resume live rounds from `active_rounds[]` or dispatch new rounds up to `max_parallel_rounds`.
4. Use one `orchestrator/round-<nn>-<slug>` branch and one `orchestrator/worktrees/<round-id>` worktree per round.
5. If a round plan authors `worker-plan.json`, launch worker implementers and an integration implementer exactly as required.
6. Move reviewed rounds through `pending-merge`, `merge`, and `update-roadmap` only when merge-order and base-branch rules allow it.
7. Re-read `state.json` plus the active roadmap bundle before deciding whether another round must start immediately.

## Load at Startup

- `orchestrator/state.json`
- Active roadmap bundle `roadmap.md` resolved from `state.json.roadmap_dir`
- Active roadmap bundle `retry-subloop.md` resolved from `state.json.roadmap_dir`
- [state-machine.md](references/state-machine.md)
- [resume-rules.md](references/resume-rules.md)

## Load Per Stage

- `select-task` / `update-roadmap`: `orchestrator/roles/guider.md`
- `plan`: `orchestrator/roles/planner.md`
- `implement`: `orchestrator/roles/implementer.md`
- `review`: `orchestrator/roles/reviewer.md`, active roadmap bundle
  `verification.md`
- `merge`: `orchestrator/roles/merger.md`
- Recovery: `orchestrator/roles/recovery-investigator.md` and
  [recovery-investigator.md](references/recovery-investigator.md)
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
- `update-roadmap` belongs to the guider.
- `done` is terminal only when there are no unfinished milestones and no live
  rounds.

Do not simulate these roles in your own voice.

## Controller Rules

- Update only machine-control state directly.
- Normalize old serial state into the current schema before scheduling.
- Require `roadmap_id`, `roadmap_revision`, and `roadmap_dir` in
  `orchestrator/state.json`; if any are missing or unusable, stop and record
  the exact controller error instead of guessing.
- Treat `roadmap_id` as an opaque scaffolded identifier, usually
  `YYYY-MM-DD-NN-<slug>`; preserve it verbatim and never recompute it from
  titles, paths, or directory names.
- Resolve live roadmap files only from `state.json.roadmap_dir`; do not use
  top-level `orchestrator/roadmap.md`, `orchestrator/verification.md`, or
  `orchestrator/retry-subloop.md` as runtime sources.
- After `update-roadmap`, re-read `orchestrator/state.json`, resolve the active
  roadmap bundle again from `roadmap_dir`, and immediately start the next round
  when unfinished `[pending]` or `[in-progress]` milestones remain and the
  concurrency cap allows it.
- Treat controller `done` as terminal only when the roadmap has no unfinished
  milestones, there are no live rounds, or a recorded controller blockage or
  user interruption lawfully stops progress.
- If the guider authored a new roadmap revision during `update-roadmap`,
  activate it by updating `state.json` `roadmap_id`, `roadmap_revision`, and
  `roadmap_dir` before the next roadmap re-check.
- See [delegation-boundaries.md](references/delegation-boundaries.md) and
  [state-machine.md](references/state-machine.md) for complete rules.

## Recovery Rules

- For any non-terminal delegated-stage failure, attempt a qualifying
  `recovery-investigator` before recording blockage in `orchestrator/state.json`.
- Record direct blockage only after `recovery-investigator` has been attempted
  and failed to produce a qualifying recovery path, or after recording a
  deterministic reason no available delegation mechanism can launch one.
- See [recovery-investigator.md](references/recovery-investigator.md) and [resume-rules.md](references/resume-rules.md) for complete recovery procedures.

## Subagent Rules

- Use a fresh real subagent for each delegated stage or worker.
- Never interrupt a live subagent.
- Never set a timeout on a live subagent.
- Wait for the active subagent to finish before continuing.
- Do not do the delegated work yourself while a role agent should own it.

## Completion

Continue until every roadmap milestone in the active bundle is complete or a
recorded controller error lawfully blocks safe progress. Do not stop just
because one round reaches `done`; terminal completion requires a roadmap
re-check confirming no unfinished milestones and no live rounds, or a lawful
recorded blockage or explicit user interruption. For non-terminal
delegated-stage failures, stop only after `recovery-investigator` has been
attempted or deterministically ruled out. For corrupt or missing state, exhaust
lawful recovery paths before recording the exact problem in `state.json`.

## Common Mistakes

- Simulating roles by writing `selection.md`, `plan.md`, or `review.md` yourself.
- Skipping `recovery-investigator` and recording blockage after one delegation failure.
- Inventing parallelism by launching rounds without roadmap metadata authorization.
- Guessing missing state by inferring `roadmap_id` from directory names.
- Treating `pending-merge` as done and proceeding before merge-order is satisfied.
- Authoring delegated artifacts by writing `implementation-notes.md` or `merge.md` yourself.

## Pre-Completion Self-Check

Before sending a final response, verify ALL of these:
- `state.json` `active_rounds` is empty
- Active roadmap bundle has no `[pending]` or `[in-progress]` milestones
- No unresolved `resume_error` or `resume_errors` entries remain
- If stopping due to blockage: precise error recorded in `state.json`
- If stopping due to user interruption: state is consistent and saved

## Operational Limits

- Maximum 3 consecutive retry attempts per round before recording blockage
- Maximum 50 tool calls per delegated stage before pausing for assessment
- If a single round has cycled through review -> plan -> implement -> review
  more than 3 times, record the pattern and ask the user

## Resources

- [state-machine.md](references/state-machine.md)
- [resume-rules.md](references/resume-rules.md)
- [recovery-investigator.md](references/recovery-investigator.md)
- [delegation-boundaries.md](references/delegation-boundaries.md)
- [worktree-merge-rules.md](references/worktree-merge-rules.md)
