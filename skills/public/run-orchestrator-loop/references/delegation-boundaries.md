# Delegation Boundaries

The runtime skill is a controller, not a worker.

## The Orchestrator May Do Directly

- read repo and orchestrator state
- read repo-local retry contract docs when present
- create the round branch and worktree
- update `orchestrator/state.json`
- record artifact paths, retry-state fields, and stage markers exactly as the repo-local contract requires
- launch and use the shared-skill-owned `recovery-investigator` for recovery diagnosis when controller-visible evidence for the active stage is missing or untrustworthy, and run controller-owned recovery repair actions directly without authoring the investigation itself
- record direct blockage only when no qualifying `recovery-investigator` can launch through any available delegation mechanism
- perform squash-merge bookkeeping after approval

## The Orchestrator Must Delegate

- task selection
- roadmap edits
- round planning
- implementation
- review decisions
- merge-note authoring
- all substantive guider/planner/implementer/reviewer/merger outputs, including retry-stage outputs

## Subagent Rules

- Use real subagents, not simulated roles.
- Use a fresh subagent for each stage.
- The `recovery-investigator` is shared-skill-owned, not repo-local.
- The `recovery-investigator` may not act as the substantive stage reviewer.
- Never interrupt a live subagent.
- Never set a timeout on a live subagent.
- Wait for the subagent to finish before continuing.

## If Real Subagents Are Unavailable

Stop and tell the user the runtime skill cannot honor its delegation contract in the current environment.
