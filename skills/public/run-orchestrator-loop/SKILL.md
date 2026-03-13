---
name: run-orchestrator-loop
description: Use when a repository already has an initialized `orchestrator/` directory and needs the delegated orchestrator loop started or resumed through guider, planner, implementer, reviewer, and merger stages.
---

# Run Orchestrator Loop

## Overview

Act as a pure controller over the persisted repo-local orchestrator contract. Read state, delegate every substantive stage to fresh real subagents, update only machine-control state, and continue until the roadmap is complete.

If real subagents are unavailable, stop and tell the user this skill cannot honor its delegation contract in the current environment.

## Workflow

1. Load state and references.
2. Resume an active round or start a new one with the guider.
3. Use one branch and one worktree for the active round.
4. Delegate each stage in order.
5. Squash-merge approved rounds and continue.

## Load Before Acting

Read these files first:

- `orchestrator/state.json`
- `orchestrator/roadmap.md`
- `orchestrator/verification.md`
- `orchestrator/roles/`
- [state-machine.md](references/state-machine.md)
- [resume-rules.md](references/resume-rules.md)
- [delegation-boundaries.md](references/delegation-boundaries.md)
- [worktree-merge-rules.md](references/worktree-merge-rules.md)

## Stage Rules

- `select-task` belongs to the guider.
- `plan` belongs to the planner.
- `implement` belongs to the implementer.
- `review` belongs to the reviewer.
- `merge` uses merger-authored notes plus controller bookkeeping.
- `update-roadmap` belongs to the guider.

Do not simulate these roles in your own voice.

## Controller Rules

- Update only machine-control state directly.
- Keep the same round id, branch, and worktree until the round is accepted.
- On review rejection, return to `plan` for the same round.
- Resume the exact incomplete stage after interruption.
- Merge only after explicit reviewer approval.

## Subagent Rules

- Use a fresh real subagent for each stage.
- Never interrupt a live subagent.
- Never set a timeout on a live subagent.
- Wait for the active subagent to finish before continuing.
- Do not do the delegated work yourself while a role agent should own it.

## Completion

Continue round by round until every roadmap item is complete or a recorded controller error blocks safe progress. When blocked by corrupt or missing state, record the exact problem in `state.json` instead of guessing.

## Resources

- [state-machine.md](references/state-machine.md): stage order, ownership, and legal transitions
- [resume-rules.md](references/resume-rules.md): automatic resume behavior
- [delegation-boundaries.md](references/delegation-boundaries.md): what the controller may and may not do
- [worktree-merge-rules.md](references/worktree-merge-rules.md): per-round worktree and squash-merge rules
