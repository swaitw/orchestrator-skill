# Delegation Boundaries

The runtime skill is a controller, not a worker.
Runtime role loading happens only from `orchestrator/roles/`.

## The Orchestrator May Do Directly

- read repo and orchestrator state
- read the active roadmap bundle and repo-local retry contract docs resolved from `state.json.roadmap_dir`
- create the round branch and worktree
- update `orchestrator/state.json`, including active roadmap metadata when a reviewed `update-roadmap` stage lawfully activates a new revision
- record artifact paths, retry-state fields, and stage markers exactly as the repo-local contract requires
- launch and use the repo-local `recovery-investigator` from `orchestrator/roles/recovery-investigator.md` for recovery diagnosis when controller-visible evidence for the active stage is missing or untrustworthy, and run controller-owned recovery repair actions directly without authoring the investigation itself
- attempt `recovery-investigator` for non-terminal delegated-stage stop
  situations before recording delegation blockage
- record the precise blockage in `orchestrator/state.json` only after an
  attempted `recovery-investigator` fails to produce a qualifying recovery
  path, or when no qualifying `recovery-investigator` can launch through any
  available delegation mechanism
- perform squash-merge bookkeeping after approval

## The Orchestrator Must Delegate

- task selection
- roadmap bundle edits
- round planning
- implementation
- review decisions
- merge-note authoring
- all substantive guider/planner/implementer/reviewer/merger outputs, including retry-stage outputs

## Subagent Rules

- Use real subagents, not simulated roles.
- Load each runtime role only from `orchestrator/roles/<role>.md`.
- Use a fresh subagent for each stage.
- The `recovery-investigator` runtime role is `orchestrator/roles/recovery-investigator.md`.
- The `recovery-investigator` may not act as the substantive stage reviewer.
- Never interrupt a live subagent.
- Never set a timeout on a live subagent.
- Wait for the subagent to finish before continuing.
- Do not convert a failed role-stage launch directly into terminal blockage
  unless `recovery-investigator` has been attempted or deterministically
  ruled out.
- If a required `orchestrator/roles/<role>.md` file is missing, stop and record the exact controller error instead of inventing one.

## If Real Subagents Are Unavailable

Record the precise blockage in `orchestrator/state.json`, then tell the user the runtime skill cannot honor its delegation contract in the current environment.
