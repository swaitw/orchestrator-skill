---
name: scaffold-orchestrator-loop
description: Use when starting a new multi-step project that needs structured orchestration, including a deep alignment brainstorm before creating `orchestrator/`, or when a terminal control plane needs a fresh roadmap family.
---

# Scaffold Orchestrator Loop

## Overview

Create or advance the repository-local control plane for the orchestrator
workflow. Review the goal and repository first, run a deep alignment
brainstorm, get explicit approval of the roadmap strategy, then either
bootstrap a tailored top-level `orchestrator/` contract from assets or open a
fresh roadmap family inside an existing terminal control plane. In both modes,
stop after the setup checkpoint commit. Do not start runtime rounds. The
contract must support milestone-level roadmap strategy, guider-extracted round
work, planner-authored worker fan-out, and safe serial defaults for
repositories that never opt into concurrency.

## Workflow

1. Survey the repository, the goal, and mode.
2. Run the alignment brainstorm and get approval.
3. Build the new active roadmap family.
4. Bootstrap or update the `orchestrator/` contract.
5. Commit the setup checkpoint and stop.

## Step 1: Survey the Repository

Inspect the repository before writing anything:

- top-level files and docs
- existing tests and verification commands
- current Git status
- whether Git is initialized already

If Git is missing, initialize it with `git init -b main`.

If `orchestrator/` already exists, also inspect:

- `orchestrator/state.json`
- `orchestrator/project-contract.md` when present
- the active roadmap bundle resolved from `roadmap_id`, `roadmap_revision`, and
  `roadmap_dir`
- `orchestrator/roadmap.md`, `orchestrator/verification.md`, and
  `orchestrator/retry-subloop.md` when they exist
- the current role files under `orchestrator/roles/`

Confirm whether the existing control plane is terminal, whether prior work is
finished, which roadmap style it uses, and whether any shared control-plane
files are missing or stale.

## Mode Resolution

Determine mode from repository state instead of asking the user for a flag.

1. If no top-level `orchestrator/` exists, use `bootstrap`.
2. If `orchestrator/` exists, parse `orchestrator/state.json`.
3. Enter `next-family` only when all of the following are true:
   - `controller_stage == "done"`
   - `active_round_id == null`
   - `active_rounds == []`
   - `pending_merge_rounds == []`
   - `retry == null`
   - the active roadmap bundle named by `roadmap_id`, `roadmap_revision`, and
     `roadmap_dir` has no unfinished milestones
4. If `orchestrator/` exists but any of those checks fail, stop with a precise
   refusal explaining that the existing control plane is still live or
   unfinished and must be resumed through `$run-orchestrator-loop` or repaired
   directly before a new family can be scaffolded.

## Step 2: Alignment Brainstorm

Read [alignment-brainstorm.md](references/alignment-brainstorm.md), then turn
the user's goal and the repository survey into an approved orchestration
strategy before drafting any roadmap files.

The alignment phase must:

- clarify outcome, non-goals, success criteria, constraints, risk, sequencing,
  and concurrency posture;
- ask only missing high-signal questions as one-question-at-a-time
  multiple-choice decisions, using a host structured choice UI/tool when it is
  available in the current mode, and text options only as a fallback;
- propose 2-3 roadmap strategies with tradeoffs and a recommendation;
- summarize selected answers in an alignment decision ledger;
- get explicit user approval of the ledger and chosen strategy;
- carry the approved alignment into `roadmap.md`, `project-contract.md`, and
  `verification.md`; and
- ensure every `project-contract.md` section has concrete entries or the exact
  phrase `none discovered yet`.

If the user already supplied a complete approved strategy, restate the durable
alignment you will scaffold and proceed only if that direction is explicit
enough to persist without guessing.

## Step 3: Build the New Active Roadmap Family

Read [roadmap-generation.md](references/roadmap-generation.md), mint a fresh
stable `roadmap_id` in `YYYY-MM-DD-NN-<slug>` form, and draft the repo-specific
roadmap content from the approved alignment for
`orchestrator/roadmaps/<roadmap_id>/rev-001/roadmap.md`. Make milestones larger
than a round, include
candidate directions for guider extraction, and keep the roadmap strategic
rather than implementation-sized. Use stable milestone and direction ids plus
explicit coordination and parallel-lane guidance. Never reopen an older roadmap
family by appending items or reusing a used revision.

## Step 4: Bootstrap Or Update the Repo Contract

Read [repo-contract.md](references/repo-contract.md) and [verification-contract.md](references/verification-contract.md).

### Mode A: `bootstrap`

- copy the full [assets/orchestrator](assets/orchestrator) scaffold tree into
  the target repo root as `orchestrator/`
- write the drafted roadmap into the new active roadmap bundle and tailor
  `verification.md` plus `retry-subloop.md` for the repo
- replace template placeholders with repo-specific content
- persist the approved alignment summary, success criteria, non-goals, and
  chosen strategy in the active roadmap bundle
- set `state.json` `roadmap_id`, `roadmap_revision`, and `roadmap_dir` to the
  initial active roadmap bundle
- set `state.json` `contract_version` and `roadmap_style`
- set `state.json` parallel-safe defaults such as `controller_stage`,
  `max_parallel_rounds`, `active_rounds`, `pending_merge_rounds`, and `retry`
- tailor `orchestrator/project-contract.md` for repo-wide invariants and keep
  roadmap revisions pointed at it instead of copying shared invariants forward
- tune the repo-local role files under `orchestrator/roles/` if the goal or
  repo needs stronger guidance
- ensure `orchestrator/worktrees/` is ignored by a tracked ignore rule so
  future rounds can use dedicated worktrees

### Mode B: `next-family`

Do not recopy the full scaffold tree over the existing repo contract.

- apply the `next-family` setup and reset rules from
  [repo-contract.md](references/repo-contract.md)
- create only the new active roadmap bundle and any missing shared contract
  files
- create or update `orchestrator/roadmaps/<roadmap_id>/roadmap-history.md`
  with compact prior-family or migration notes when needed
- preserve prior roadmap families, prior rounds, worktrees, and role files by
  default
- refresh optional top-level pointer files only when they already exist

For `next-family`, reset `state.json` to an idle-but-runnable state for the
new family using the reset rules in [repo-contract.md](references/repo-contract.md).

Keep `state.json` machine-oriented. Put reasoning and reviewable content in the
human-facing files under the active roadmap bundle and round artifacts under
`orchestrator/`. If worker fan-out is later used for a round, the planner must
author machine-readable `worker-plan.json` beside the human-facing round
artifacts rather than smuggling worker state into prose.

## Guardrails

- Do not open a new family while the existing orchestrator is non-terminal.
- Do not draft or write a roadmap family before the alignment strategy is
  approved.
- Do not rewrite a roadmap revision that has already been used by rounds.
- Do not delete prior families, prior revisions, or prior round artifacts.
- Do not blindly recopy scaffold assets over an existing customized
  `orchestrator/`.
- Do not silently widen from opening a bounded fresh family into retuning the
  entire shared orchestrator contract unless the user explicitly asked for that
  broader refresh.

## Step 5: Create the Checkpoint Commit

After the setup files are tailored:

- stage only the scaffolded or updated control-plane files plus any ignore-file
  edits required for `orchestrator/worktrees/`
- create the setup checkpoint commit
- stop after setup

Do not start implementation rounds. Runtime orchestration belongs to `$run-orchestrator-loop`, not this skill.

## Resources

- [alignment-brainstorm.md](references/alignment-brainstorm.md): required
  scaffold-time alignment and approval gate
- [repo-contract.md](references/repo-contract.md): required file layout, reset rules, and ownership rules
- [roadmap-generation.md](references/roadmap-generation.md): how to derive the initial roadmap from the goal and repo
- [verification-contract.md](references/verification-contract.md): how to tailor the active roadmap bundle's `verification.md`
- [assets/orchestrator](assets/orchestrator): templates for the scaffolded controller state, role files, round artifacts, and worktree directory
