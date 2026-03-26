---
name: scaffold-orchestrator-loop
description: Use when a repository needs a new repo-local orchestrator workflow scaffolded from a goal, including an initial roadmap, `orchestrator/` control files, a verification contract, and repo-local `orchestrator/roles/` definitions before delegated runtime execution begins.
---

# Scaffold Orchestrator Loop

## Overview

Create the repository-local control plane for the orchestrator workflow. Review the goal and repository first, scaffold a tailored `orchestrator/` directory containing controller state, repo-local role files under `orchestrator/roles/`, the initial revisioned roadmap bundle under `orchestrator/roadmaps/`, round artifacts under `orchestrator/rounds/`, and per-round worktree preparation under `orchestrator/worktrees/`, then stop after the initial checkpoint commit. The scaffolded contract must support explicit parallel-safe roadmap items, planner-authored worker fan-out, and safe serial defaults for repositories that never opt into concurrency.

## Workflow

1. Survey the repository and the goal.
2. Build the initial roadmap.
3. Scaffold and tailor the `orchestrator/` contract.
4. Commit the initial checkpoint and stop.

## Step 1: Survey the Repository

Inspect the repository before writing anything:

- top-level files and docs
- existing tests and verification commands
- current Git status
- whether Git is initialized already

If Git is missing, initialize it with `git init -b main`.

## Step 2: Build the Initial Roadmap

Read [roadmap-generation.md](references/roadmap-generation.md), mint a stable `roadmap_id` in `YYYY-MM-DD-NN-<slug>` form, and draft the repo-specific ordered roadmap content for `orchestrator/roadmaps/<roadmap_id>/rev-001/roadmap.md`. Keep later items coarse and make the next item concrete. Each item must include a stable `Item id:` plus explicit parallel metadata, even when the item remains serial.

## Step 3: Scaffold the Repo Contract

Read [repo-contract.md](references/repo-contract.md) and [verification-contract.md](references/verification-contract.md).

Then:

- copy the full [assets/orchestrator](assets/orchestrator) scaffold tree into the target repo root as `orchestrator/` so `roles/`, `rounds/`, and `worktrees/` are created together
- write the drafted roadmap into the scaffolded active roadmap bundle and tailor `verification.md` plus `retry-subloop.md` for the repo
- replace template placeholders with repo-specific content
- set `state.json` `roadmap_id`, `roadmap_revision`, and `roadmap_dir` to the initial active roadmap bundle
- set `state.json` parallel-safe defaults such as `controller_stage`, `max_parallel_rounds`, `active_rounds`, and `pending_merge_rounds`
- tune the repo-local role files under `orchestrator/roles/` if the goal or repo needs stronger guidance
- ensure `orchestrator/worktrees/` is ignored by a tracked ignore rule so future rounds can use dedicated worktrees

Keep `state.json` machine-oriented. Put reasoning and reviewable content in the human-facing files under the active roadmap bundle and round artifacts under `orchestrator/`. If worker fan-out is later used for a round, the planner must author machine-readable `worker-plan.json` beside the human-facing round artifacts rather than smuggling worker state into prose.

## Step 4: Create the Checkpoint Commit

After the scaffolded files are tailored:

- stage only the new `orchestrator/` files and any ignore-file edits required for `orchestrator/worktrees/`
- create the initial checkpoint commit
- stop after setup

Do not start implementation rounds. Runtime orchestration belongs to `$run-orchestrator-loop`, not this skill.

## Resources

- [repo-contract.md](references/repo-contract.md): required file layout, state schema, and ownership rules
- [roadmap-generation.md](references/roadmap-generation.md): how to derive the initial roadmap from the goal and repo
- [verification-contract.md](references/verification-contract.md): how to tailor the active roadmap bundle's `verification.md`
- [assets/orchestrator](assets/orchestrator): templates for the scaffolded controller state, role files, round artifacts, and worktree directory
