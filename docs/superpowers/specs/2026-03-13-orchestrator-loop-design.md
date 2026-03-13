# Orchestrator Loop Skill Set Design

## Summary

Build a two-skill system for repository-local orchestration:

- `scaffold-orchestrator-loop` sets up the roadmap, persistent loop files, role prompts, verification contract, and initial Git checkpoint.
- `run-orchestrator-loop` acts only as a coordinator over that persisted contract and delegates all real work to subagents.

The design is optimized for strict delegation, resumability, and clean audit history.

## Goals

- Accept a high-level goal and turn it into an executable roadmap.
- Run implementation in a strict orchestrator pattern where the orchestrator never performs the substantive work itself.
- Persist enough state in the repo to resume after interruption without ambiguity.
- Use one linear round at a time so resume, review, and merge behavior stay simple.
- Keep role behavior inspectable and easy to tune through repo-local role prompt files.

## Non-Goals

- Parallel execution inside a single roadmap round.
- Long-lived role agents that carry context across rounds.
- Orchestrator-authored plans, code, reviews, or merge notes.
- Hidden state that only exists in conversation context.

## Approach Options Considered

### Option 1: Thin setup plus implicit runtime behavior

Use a setup skill to create a minimal roadmap and let the runtime skill carry most behavior in its own prompt.

Trade-offs:

- Lower upfront file count.
- Harder to inspect and tune.
- More resume logic stays implicit in the orchestrator.

### Option 2: Two-skill control plane with a repo contract

Use one setup skill to create a full repository-local contract and one runtime skill to execute a state machine over that contract.

Trade-offs:

- Clear persistence and resume model.
- Easy to audit and modify.
- Slightly more upfront scaffolding.

This is the recommended approach.

### Option 3: Full framework with separate packaged role skills

Build setup and runtime skills plus separate reusable packaged skills for each role.

Trade-offs:

- Highest modularity and reuse.
- More packaging and maintenance overhead before the loop is proven.

## Recommended Architecture

### Skill 1: `scaffold-orchestrator-loop`

Responsibilities:

- Review the user goal and the current repository.
- Initialize Git when the target directory is not already a repository.
- Create the first concrete roadmap.
- Create the persistent `orchestrator/` contract.
- Scaffold inspectable role prompt files.
- Define the canonical verification contract.
- Create an initial checkpoint commit so future worktrees are available.

Outputs:

- `orchestrator/roadmap.md`
- `orchestrator/state.json`
- `orchestrator/verification.md`
- `orchestrator/roles/*.md`
- `orchestrator/rounds/`

### Skill 2: `run-orchestrator-loop`

Responsibilities:

- Read the persisted `orchestrator/` contract.
- Resume the active round if one exists.
- Otherwise dispatch task selection to the guider.
- Create exactly one branch and one worktree for the current round.
- Spawn fresh planner, implementer, reviewer, and merger agents as needed for the current stage.
- Loop within the same round until reviewer approval.
- Squash-merge the accepted round into the main branch.
- Update machine-control state and continue until the roadmap is complete.

Constraints:

- Never perform substantive implementation work directly.
- Never write plans, code, reviews, or merge notes itself.
- Only update machine-control state such as stage markers, round metadata, status, and resume pointers.
- Never use long-lived role agents across rounds.

## Repository Contract

All persistent loop state lives in a visible top-level `orchestrator/` directory.

### Required Files

- `orchestrator/roadmap.md`
  - Ordered roadmap items.
  - Status per item.
  - Dependencies and completion notes.

- `orchestrator/state.json`
  - Active round id.
  - Current stage.
  - Branch and worktree metadata.
  - Resume pointers.
  - Current task selection.

- `orchestrator/verification.md`
  - Canonical verification contract.
  - Baseline commands or checks every reviewer must run.
  - Guidance for adding change-specific checks.

- `orchestrator/roles/guider.md`
- `orchestrator/roles/planner.md`
- `orchestrator/roles/implementer.md`
- `orchestrator/roles/reviewer.md`
- `orchestrator/roles/merger.md`

### Round Artifacts

Each round gets a dedicated folder:

- `orchestrator/rounds/<round-id>/`

That folder holds delegated artifacts only, for example:

- `selection.md`
- `plan.md`
- `implementation-notes.md`
- `review.md`
- `merge.md`

The orchestrator may record which artifact paths exist, but it does not author their substantive contents.

## Role Ownership

### Orchestrator

The orchestrator is a pure coordinator. It is responsible for:

- Reading persisted state.
- Dispatching the appropriate role for the current stage.
- Recording the result in machine-control state.
- Advancing or rewinding the state machine.
- Managing branch, worktree, and merge bookkeeping.

It is not responsible for selecting tasks, planning, implementing, reviewing, or authoring merge notes.

### Guider

The guider owns `select-task`.

The guider is responsible for:

- Reviewing `orchestrator/roadmap.md`.
- Reviewing current repository and prior round status.
- Choosing the next roadmap item to execute.
- Recording rationale in a delegated round artifact.
- Recommending roadmap updates after accepted rounds.

### Planner

The planner is responsible for:

- Turning the selected task into a concrete executable plan.
- Revising the plan after rejected reviews.
- Staying within the current round until reviewer approval is achieved.

### Implementer

The implementer is responsible for:

- Executing the planner's approved plan inside the round worktree.
- Producing the implementation artifact trail needed for review.

### Reviewer

The reviewer is responsible for:

- Reviewing the round changes.
- Running the canonical verification contract plus task-specific checks.
- Rejecting or approving the round.
- Producing explicit review feedback as a delegated artifact.

### Merger

The merger is responsible for:

- Preparing the accepted round for squash merge.
- Producing merge notes as a delegated artifact.

The orchestrator performs merge bookkeeping, but the merger owns the delegated merge content.

## Round Lifecycle

Each round is strictly linear and uses one dedicated branch and one dedicated worktree.

State machine:

1. `select-task`
2. `plan`
3. `implement`
4. `review`
5. `merge`
6. `update-roadmap`
7. `done`

Rules:

- Only one round may be active at a time.
- Fresh stage-specific subagents are created for each stage and each round.
- Planner, implementer, and reviewer operate against the same round worktree.
- No parallel sub-tasks are allowed inside a round.
- Review rejection returns the round to plan, then implement, then review again.
- There is no retry cap inside a round.

## Resume Model

Resume is automatic.

On startup, the runtime skill:

- Reads `orchestrator/state.json`.
- Detects whether there is an active round.
- Re-opens the recorded round worktree and branch.
- Restarts from the last incomplete stage using a fresh subagent for that stage.

Rules:

- If interruption happens mid-stage, resume that exact stage.
- If review rejects a round, keep the same round id, branch, and worktree.
- Do not create a new round unless the prior round reaches `done`.
- Do not ask the user for resume confirmation by default.

## Git And Merge Model

- Initialize Git during setup if it does not already exist.
- Create an initial checkpoint commit after scaffolding the repo contract.
- Use one dedicated branch per round.
- After reviewer approval, squash-merge the round branch into the main branch.
- Archive or mark the round as completed after merge.

Rationale:

- One roadmap item becomes one atomic accepted change.
- History remains readable.
- Resume behavior stays simple because the round branch persists until acceptance.

## Verification Contract

The setup skill must write a canonical verification contract into `orchestrator/verification.md`.

That contract defines:

- The required baseline checks for every round.
- How reviewers record verification outcomes.
- When change-specific checks must be added.

Reviewer approval requires:

- Successful execution of the canonical checks.
- Successful execution of any task-specific checks introduced by the change.
- A delegated approval artifact for the round.

## Error Handling

- Missing Git repo: initialize Git during setup.
- Missing worktree on resume: reconstruct from recorded branch metadata when possible; otherwise fail loudly in control-state and require human intervention.
- Rejected review: planner revises plan, then return to implement and review within the same round.
- Corrupt state file: stop the runtime loop and record a recoverable orchestrator error rather than guessing.

## Testing Strategy

The eventual implementation should verify:

- Setup on an empty non-Git directory.
- Setup on an existing Git repository.
- Runtime start from a fresh scaffolded state.
- Resume from each stage marker.
- Review rejection and in-round replan loop.
- Accepted round squash-merge.
- Roadmap completion behavior.

## Open Implementation Notes

- The state schema should stay small and machine-oriented; human-readable reasoning belongs in delegated round artifacts.
- Role prompt files should remain editable by users so the orchestrator team can be tuned without rewriting the core skill.
- The runtime prompt should emphasize that the orchestrator may only update control-state files.
