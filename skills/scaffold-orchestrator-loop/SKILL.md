---
name: scaffold-orchestrator-loop
description: Use when starting a new multi-step project that needs structured orchestration and the repository either has no `orchestrator/` yet or has a terminal control plane that needs a fresh roadmap family.
---

# Scaffold Orchestrator Loop

## Overview

Create or advance the repository-local control plane for the orchestrator
workflow. Review the goal and repository first, then either bootstrap a
tailored top-level `orchestrator/` contract from assets or open a fresh
roadmap family inside an existing terminal control plane. In both modes, stop
after the setup checkpoint commit. Do not start runtime rounds. The contract
must support milestone-level roadmap strategy, guider-extracted round work,
planner-authored worker fan-out, and safe serial defaults for repositories
that never opt into concurrency.

## Workflow

1. Survey the repository, the goal, and mode.
2. Build the new active roadmap family.
3. Bootstrap or update the `orchestrator/` contract.
4. Commit the setup checkpoint and stop.

## Step 1: Survey the Repository

Inspect the repository before writing anything:

- top-level files and docs
- existing tests and verification commands
- current Git status
- whether Git is initialized already

If Git is missing, initialize it with `git init -b main`.

If `orchestrator/` already exists, also inspect:

- `orchestrator/state.json`
- the active roadmap bundle resolved from `roadmap_id`, `roadmap_revision`, and
  `roadmap_dir`
- `orchestrator/roadmap.md`, `orchestrator/verification.md`, and
  `orchestrator/retry-subloop.md` when they exist
- the current role files under `orchestrator/roles/`

Confirm whether the existing control plane is terminal, whether prior work is
finished, and whether any shared control-plane files are missing or stale.

## Mode Resolution

Determine mode from repository state instead of asking the user for a flag.

1. If no top-level `orchestrator/` exists, use `bootstrap`.
2. If `orchestrator/` exists, parse `orchestrator/state.json`.
3. Enter `next-family` only when all of the following are true:
   - `stage == "done"`
   - `controller_stage == "done"`
   - `active_round_id == null`
   - `active_rounds == []`
   - `pending_merge_rounds == []`
   - `retry == null`
   - the active roadmap bundle named by `roadmap_id`, `roadmap_revision`, and
     `roadmap_dir` contains no `[pending]` or `[in-progress]` items
4. If `orchestrator/` exists but any of those checks fail, stop with a precise
   refusal explaining that the existing control plane is still live or
   unfinished and must be resumed through `$run-orchestrator-loop` or repaired
   directly before a new family can be scaffolded.

Do not guess. The roadmap-content check is required so stale machine state does
not silently open a new family on top of unfinished prior work.

## Step 2: Build the New Active Roadmap Family

Read [roadmap-generation.md](references/roadmap-generation.md), mint a fresh
stable `roadmap_id` in `YYYY-MM-DD-NN-<slug>` form, and draft the repo-specific
strategy-backlog roadmap content for
`orchestrator/roadmaps/<roadmap_id>/rev-001/roadmap.md`. Make milestones larger
than a round, include candidate directions for guider extraction, and keep the
roadmap strategic rather than implementation-sized. Use stable milestone and
direction ids plus explicit coordination and parallel-lane guidance. Never
reopen an older roadmap family by appending items or reusing a used revision.

## Step 3: Bootstrap Or Update the Repo Contract

Read [repo-contract.md](references/repo-contract.md) and [verification-contract.md](references/verification-contract.md).

### Mode A: `bootstrap`

- copy the full [assets/orchestrator](assets/orchestrator) scaffold tree into
  the target repo root as `orchestrator/`
- write the drafted roadmap into the new active roadmap bundle and tailor
  `verification.md` plus `retry-subloop.md` for the repo
- replace template placeholders with repo-specific content
- set `state.json` `roadmap_id`, `roadmap_revision`, and `roadmap_dir` to the
  initial active roadmap bundle
- set `state.json` parallel-safe defaults such as `controller_stage`,
  `max_parallel_rounds`, `active_rounds`, `pending_merge_rounds`, and `retry`
- tune the repo-local role files under `orchestrator/roles/` if the goal or
  repo needs stronger guidance
- ensure `orchestrator/worktrees/` is ignored by a tracked ignore rule so
  future rounds can use dedicated worktrees

### Mode B: `next-family`

Do not recopy the full scaffold tree over the existing repo contract.

- reuse the existing `orchestrator/roles/`, `orchestrator/rounds/`, and
  `orchestrator/worktrees/` directories
- scaffold only missing shared control-plane files if the repo contract is
  incomplete
- create a fresh active bundle under
  `orchestrator/roadmaps/<new-roadmap-id>/rev-001/`
- update `orchestrator/state.json` to point at the new active bundle
- refresh `orchestrator/roadmap.md`, `orchestrator/verification.md`, and
  `orchestrator/retry-subloop.md` when those pointer files already exist
- preserve role files by default unless they are missing, the user explicitly
  wants a shared-contract refresh, or repo-local wording is clearly
  incompatible with the new goal and the change is part of setup

For `next-family`, reset `state.json` to an idle-but-runnable state for the
new family:

- preserve `base_branch`
- set the new `roadmap_id`, `roadmap_revision`, and `roadmap_dir`
- set `stage: "select-task"`
- set `controller_stage: "dispatch-rounds"`
- set `active_round_id: null`
- set `active_rounds: []`
- set `pending_merge_rounds: []`
- clear `current_task`, `branch`, `worktree_path`, `active_round_dir`, and
  `round_artifacts`
- preserve `last_completed_round` as historical data
- clear `resume_error`, `resume_errors`, and `retry`

Keep `state.json` machine-oriented. Put reasoning and reviewable content in the
human-facing files under the active roadmap bundle and round artifacts under
`orchestrator/`. If worker fan-out is later used for a round, the planner must
author machine-readable `worker-plan.json` beside the human-facing round
artifacts rather than smuggling worker state into prose.

## Guardrails

- Do not open a new family while the existing orchestrator is non-terminal.
- Do not rewrite a roadmap revision that has already been used by rounds.
- Do not delete prior families, prior revisions, or prior round artifacts.
- Do not blindly recopy scaffold assets over an existing customized
  `orchestrator/`.
- Do not silently widen from opening a bounded fresh family into retuning the
  entire shared orchestrator contract unless the user explicitly asked for that
  broader refresh.

## Step 4: Create the Checkpoint Commit

After the setup files are tailored:

- stage only the scaffolded or updated control-plane files plus any ignore-file
  edits required for `orchestrator/worktrees/`
- create the setup checkpoint commit
- stop after setup

Do not start implementation rounds. Runtime orchestration belongs to `$run-orchestrator-loop`, not this skill.

## Resources

- [repo-contract.md](references/repo-contract.md): required file layout, state schema, and ownership rules
- [roadmap-generation.md](references/roadmap-generation.md): how to derive the initial roadmap from the goal and repo
- [verification-contract.md](references/verification-contract.md): how to tailor the active roadmap bundle's `verification.md`
- [assets/orchestrator](assets/orchestrator): templates for the scaffolded controller state, role files, round artifacts, and worktree directory
