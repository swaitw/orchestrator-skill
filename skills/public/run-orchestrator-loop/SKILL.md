---
name: run-orchestrator-loop
description: Use when a repository already has an initialized `orchestrator/` directory and needs the delegated orchestrator loop started or resumed through guider, planner, implementer, reviewer, and merger stages.
---

# Run Orchestrator Loop

## Overview

Act as a pure controller over the persisted repo-local orchestrator contract. Read state, load any repo-local retry rules, resolve repo-local role agents, delegate every substantive stage to fresh real subagents, update only machine-control state, and continue until the roadmap is complete.

Treat incidental delegation failure as missing or untrustworthy stage artifacts that leave the controller without trustworthy controller-visible evidence for the active stage outcome.

If incidental delegation failure occurs and no available delegation mechanism can launch a qualifying `recovery-investigator`, record the precise blockage in `orchestrator/state.json`, then tell the user this skill cannot honor its delegation contract in the current environment.

## Workflow

1. Load state and references.
2. Resume an active round or start a new one with the guider.
3. Use one branch and one worktree for the active round.
4. Delegate each stage in order, including same-round retry loops required by the repo-local review contract.
5. Squash-merge approved rounds, run `update-roadmap`, then re-read the roadmap before deciding whether another round must start immediately.

## Load Before Acting

Read these files first:

- `orchestrator/state.json`
- `orchestrator/roadmap.md`
- `orchestrator/verification.md`
- `orchestrator/retry-subloop.md` when present
- `.codex/agents/` when present
- `orchestrator/roles/` only as a per-role compatibility fallback when a matching repo-local agent file is missing
- [state-machine.md](references/state-machine.md)
- [resume-rules.md](references/resume-rules.md)
- [recovery-investigator.md](references/recovery-investigator.md)
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
- For each stage, prefer the matching repo-local `.codex/agents/orchestrator-<role>.toml` file when it exists.
- If the matching repo-local agent file is missing, fall back to `orchestrator/roles/<role>.md`.
- Recovery must resolve repo-local guider/planner/implementer/reviewer/merger sources exactly as it does today.
- Keep the same round id, branch, and worktree until the round is finalized.
- When repo-local review output requests retry, return to `plan` for the same round instead of merging, even if the attempt itself was accepted as valid evidence.
- Resume the exact incomplete stage and exact retry attempt after interruption.
- Merge only after explicit reviewer approval that finalizes the current stage or round under the repo-local contract.
- Do not invent retry behavior; read it from repo-local state and retry docs when they exist.
- During recovery, the controller may take broad controller-owned recovery actions such as re-reading state, inspecting controller-visible evidence, recreating or reopening the recorded worktree, relaunching delegation, and recording controller-owned recovery notes, but it may not author guider/planner/implementer/reviewer/merger artifacts.
- After `update-roadmap`, re-read `orchestrator/roadmap.md` and immediately start the next round when unfinished `[pending]` or `[in-progress]` items remain.
- Treat `stage: "done"` as terminal only when the roadmap has no unfinished items, or when a recorded controller blockage or explicit user interruption lawfully stops progress.
- Before stopping or sending a final response, verify one of:
  - `orchestrator/roadmap.md` has no unfinished `[pending]` or `[in-progress]` items;
  - `orchestrator/state.json` records a precise controller blockage or `resume_error` that prevents safe progress; or
  - the user explicitly interrupted or redirected the loop.
- If neither repo-local role source exists for a required stage, stop and record the exact controller error instead of guessing.

## Recovery Rules

- An available delegation mechanism is any mechanism that can launch a fresh real subagent in the current environment, keep it running without timeout or interruption, and preserve the repo-local role-source contract.
- When incidental delegation failure occurs, launch a dedicated real-subagent `recovery-investigator` using [recovery-investigator.md](references/recovery-investigator.md).
- The `recovery-investigator` may recommend retrying with the same mechanism or switching to another available real-subagent mechanism.
- Record the precise direct blockage in `orchestrator/state.json` immediately only when no qualifying recovery-investigator can launch through any available delegation mechanism.
- Before the controller leaves recovery, re-check that the expected stage artifact exists and matches the current round, stage, retry attempt, and repo-local contract.
- Record terminal blockage only after lawful recovery paths are exhausted.

## Subagent Rules

- Use a fresh real subagent for each stage.
- Never interrupt a live subagent.
- Never set a timeout on a live subagent.
- Wait for the active subagent to finish before continuing.
- Do not do the delegated work yourself while a role agent should own it.

## Completion

Continue round by round until every roadmap item is complete or a recorded controller error blocks safe progress. Do not stop merely because the current stage reads `done`; `done` is terminal only after a roadmap re-check confirms there are no unfinished items, or after a lawful recorded blockage or explicit user interruption. When blocked by corrupt or missing state, exhaust lawful recovery paths first, then record the exact problem in `state.json` instead of guessing.

## Resources

- [state-machine.md](references/state-machine.md): stage order, ownership, and legal transitions
- [resume-rules.md](references/resume-rules.md): automatic resume behavior
- [recovery-investigator.md](references/recovery-investigator.md): dedicated recovery subagent contract
- [delegation-boundaries.md](references/delegation-boundaries.md): what the controller may and may not do
- [worktree-merge-rules.md](references/worktree-merge-rules.md): per-round worktree and squash-merge rules
