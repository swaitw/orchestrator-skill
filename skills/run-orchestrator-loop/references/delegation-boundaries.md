# Delegation Boundaries

The runtime skill is a controller, not a worker.
Runtime role loading happens only from `orchestrator/roles/`.

## The Orchestrator May Do Directly

- read repo and orchestrator state
- read the active roadmap bundle and repo-local retry contract docs resolved
  from `state.json.roadmap_dir`
- normalize legacy serial state into the current machine schema
- create round branches and round worktrees
- create worker branches and worker worktrees when `worker-plan.json` requires
  them
- update `orchestrator/state.json`, including active roadmap metadata when a
  reviewed `update-roadmap` stage lawfully activates a new revision
- record artifact paths, retry-state fields, pending-merge fields, worker state
  fields, and stage markers exactly as the repo-local contract requires
- resolve live round artifact paths against the recorded round
  `worktree_path`, using the parent checkout only for archived post-merge
  artifacts
- launch and use the repo-local `recovery-investigator` from
  `orchestrator/roles/recovery-investigator.md` for recovery diagnosis when
  controller-visible evidence for the active stage is missing or untrustworthy,
  and run controller-owned recovery repair actions directly without authoring
  the investigation itself
- re-read the recorded canonical round worktree and treat it as the primary
  observation surface for round-owned stage artifacts during recovery
- clear stale blockage bookkeeping, refresh artifact-path bookkeeping, and
  recreate missing worktrees when those controller-owned repairs can recover an
  already-produced stage result without authoring new stage content
- attempt `recovery-investigator` for non-terminal delegated-stage stop
  situations before recording delegation blockage
- record the precise blockage in `orchestrator/state.json` only after an
  attempted `recovery-investigator` and the full recovery ladder fail to
  produce a qualifying recovery path, or when no qualifying
  `recovery-investigator` can launch through any available delegation
  mechanism
- perform squash-merge bookkeeping after approval

## The Orchestrator Must Delegate

- task selection
- roadmap bundle edits
- round planning
- worker planning via `worker-plan.json`
- implementation
- integration implementation when worker fan-out is active
- review decisions
- merge-note authoring
- all substantive guider/planner/implementer/reviewer/merger outputs, including
  retry-stage outputs

## Subagent Rules

- Use real subagents, not simulated roles.
- Load each runtime role only from `orchestrator/roles/<role>.md`.
- Use a fresh subagent for each delegated stage or worker.
- The `recovery-investigator` runtime role is
  `orchestrator/roles/recovery-investigator.md`.
- The `recovery-investigator` may not act as the substantive stage reviewer.
- Never interrupt a live subagent.
- Never set a timeout on a live subagent.
- Wait for the subagent to finish before continuing.
- Do not convert a failed role-stage launch directly into terminal blockage
  unless `recovery-investigator` has been attempted or deterministically ruled
  out, and no same-round recovery, re-observation, or different-mechanism
  re-dispatch remains lawful.
- If a required `orchestrator/roles/<role>.md` file is missing, stop and record
  the exact controller error instead of inventing one.

## The Orchestrator May Not Do

- author `selection.md`, `plan.md`, `worker-plan.json`, `implementation-notes.md`,
  `review.md`, `review-record.json`, or `merge.md`
- choose worker ownership boundaries on its own
- perform substantive code integration or refresh itself
- approve work without a round-level reviewer

## If Real Subagents Are Unavailable

Record the precise blockage in `orchestrator/state.json`, then tell the user
the runtime skill cannot honor its delegation contract in the current
environment.
