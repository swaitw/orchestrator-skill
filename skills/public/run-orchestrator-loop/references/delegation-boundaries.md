# Delegation Boundaries

The runtime skill is a controller, not a worker.

## The Orchestrator May Do Directly

- read repo and orchestrator state
- create the round branch and worktree
- update `orchestrator/state.json`
- record artifact paths and stage markers
- perform squash-merge bookkeeping after approval

## The Orchestrator Must Delegate

- task selection
- roadmap edits
- round planning
- implementation
- review decisions
- merge-note authoring

## Subagent Rules

- Use real subagents, not simulated roles.
- Use a fresh subagent for each stage.
- Never interrupt a live subagent.
- Never set a timeout on a live subagent.
- Wait for the subagent to finish before continuing.

## If Real Subagents Are Unavailable

Stop and tell the user the runtime skill cannot honor its delegation contract in the current environment.
