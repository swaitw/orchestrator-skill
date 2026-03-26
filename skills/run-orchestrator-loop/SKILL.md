---
name: run-orchestrator-loop
description: Use when a repository already has an initialized `orchestrator/` directory and needs the delegated orchestrator loop started or resumed through guider, planner, implementer, reviewer, and merger stages.
---

# Run Orchestrator Loop

## Overview

Act as a pure controller over the persisted repo-local orchestrator contract. Read `orchestrator/state.json`, resolve the active roadmap bundle from `roadmap_id`, `roadmap_revision`, and `roadmap_dir`, load any repo-local retry rules from that bundle, resolve repo-local role agents, delegate every substantive stage to fresh real subagents, update only machine-control state, and continue until the roadmap is complete.

Treat incidental delegation failure as missing or untrustworthy stage artifacts that leave the controller without trustworthy controller-visible evidence for the active stage outcome.

If incidental delegation failure occurs, the controller must first attempt a
qualifying `recovery-investigator` before it may stop on that failure. Only
after an attempted `recovery-investigator` fails, or after the controller
records a deterministic reason why no available delegation mechanism can
launch one, may the controller record precise blockage in
`orchestrator/state.json` and tell the user this skill cannot honor its
delegation contract in the current environment.

## Workflow

1. Load state and references.
2. Resume an active round or start a new one with the guider.
3. Use one branch and one worktree for the active round.
4. Delegate each stage in order, including same-round retry loops required by the repo-local review contract.
5. Squash-merge approved rounds, run `update-roadmap`, then re-read `state.json` plus the active roadmap bundle before deciding whether another round must start immediately.

## Load Before Acting

Read these files first:

- `orchestrator/state.json`
- active roadmap bundle `roadmap.md` resolved from `state.json.roadmap_dir`
- active roadmap bundle `verification.md` resolved from `state.json.roadmap_dir`
- active roadmap bundle `retry-subloop.md` resolved from `state.json.roadmap_dir`
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
- Require `roadmap_id`, `roadmap_revision`, and `roadmap_dir` in `orchestrator/state.json`; if any are missing or unusable, stop and record the exact controller error instead of guessing.
- Treat `roadmap_id` as an opaque scaffolded identifier that should normally look like `YYYY-MM-DD-NN-<slug>`; preserve it verbatim and do not recompute it from roadmap titles or paths.
- Do not treat top-level `orchestrator/roadmap.md`, `orchestrator/verification.md`, or `orchestrator/retry-subloop.md` as live sources in this contract.
- For each stage, prefer the matching repo-local `.codex/agents/orchestrator-<role>.toml` file when it exists.
- If the matching repo-local agent file is missing, fall back to `orchestrator/roles/<role>.md`.
- Recovery must resolve repo-local guider/planner/implementer/reviewer/merger sources exactly as it does today.
- Keep the same round id, branch, and worktree until the round is finalized.
- When repo-local review output requests retry, return to `plan` for the same round instead of merging, even if the attempt itself was accepted as valid evidence.
- Resume the exact incomplete stage and exact retry attempt after interruption.
- Merge only after explicit reviewer approval that finalizes the current stage or round under the repo-local contract.
- Do not invent retry behavior; read it from repo-local state and the active roadmap bundle `retry-subloop.md`.
- During recovery, the controller may take broad controller-owned recovery actions such as re-reading state, inspecting controller-visible evidence, recreating or reopening the recorded worktree, relaunching delegation, and recording controller-owned recovery notes, but it may not author guider/planner/implementer/reviewer/merger artifacts.
- After `update-roadmap`, re-read `orchestrator/state.json`, resolve the active roadmap bundle again from `roadmap_dir`, and immediately start the next round when unfinished `[pending]` or `[in-progress]` items remain.
- Treat `stage: "done"` as terminal only when the roadmap has no unfinished items, or when a recorded controller blockage or explicit user interruption lawfully stops progress.
- Do not treat a delegated-stage failure as a lawful stop merely because the
  first role subagent returned unusable output or produced no stage artifact.
  For any non-terminal stop caused by missing or untrustworthy delegated-stage
  output, attempt `recovery-investigator` first.
- Before stopping or sending a final response, verify one of:
  - the active roadmap bundle `roadmap.md` has no unfinished `[pending]` or `[in-progress]` items;
  - `orchestrator/state.json` records a precise controller blockage or `resume_error` that prevents safe progress; or
  - the user explicitly interrupted or redirected the loop.
- If the guider authored a new roadmap revision during `update-roadmap`, activate it by updating `state.json` `roadmap_id`, `roadmap_revision`, and `roadmap_dir` before the next roadmap re-check.
- If neither repo-local role source exists for a required stage, stop and record the exact controller error instead of guessing.

## Recovery Rules

- An available delegation mechanism is any mechanism that can launch a fresh real subagent in the current environment, keep it running without timeout or interruption, and preserve the repo-local role-source contract.
- When incidental delegation failure occurs, launch a dedicated real-subagent
  `recovery-investigator` using
  [recovery-investigator.md](references/recovery-investigator.md) before
  recording a delegation-failure stop.
- The `recovery-investigator` may recommend retrying with the same mechanism or switching to another available real-subagent mechanism.
- Record the precise direct blockage in `orchestrator/state.json` only after
  the controller has either:
  - attempted `recovery-investigator` and still lacks a qualifying recovery
    result; or
  - recorded a deterministic reason why no available delegation mechanism can
    launch a qualifying `recovery-investigator`.
- The controller must not infer `recovery-investigator` unavailability solely
  from the failed role-stage launch itself.
- Before the controller leaves recovery, re-check that the expected stage artifact exists and matches the current round, stage, retry attempt, active roadmap identity, and repo-local contract.
- Record terminal blockage only after lawful recovery paths are exhausted.

## Subagent Rules

- Use a fresh real subagent for each stage.
- Never interrupt a live subagent.
- Never set a timeout on a live subagent.
- Wait for the active subagent to finish before continuing.
- Do not do the delegated work yourself while a role agent should own it.

## Completion

Continue round by round until every roadmap item in the active roadmap bundle
is complete or a recorded controller error blocks safe progress. Do not stop
merely because the current stage reads `done`; `done` is terminal only after a
roadmap re-check confirms there are no unfinished items, or after a lawful
recorded blockage or explicit user interruption. For non-terminal delegated
stage failures, a stop is lawful only after `recovery-investigator` has been
attempted or deterministically ruled out as unavailable. When blocked by
corrupt or missing state, exhaust lawful recovery paths first, then record the
exact problem in `state.json` instead of guessing.

## Resources

- [state-machine.md](references/state-machine.md): stage order, ownership, and legal transitions
- [resume-rules.md](references/resume-rules.md): automatic resume behavior
- [recovery-investigator.md](references/recovery-investigator.md): dedicated recovery subagent contract
- [delegation-boundaries.md](references/delegation-boundaries.md): what the controller may and may not do
- [worktree-merge-rules.md](references/worktree-merge-rules.md): per-round worktree and squash-merge rules
