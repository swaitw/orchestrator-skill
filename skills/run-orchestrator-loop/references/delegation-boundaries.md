# Delegation Boundaries

The runtime skill is a controller, not a worker.
Runtime role loading happens only from `orchestrator/roles/`.

## The Orchestrator May Do Directly

- read repo and orchestrator state
- read the active roadmap bundle and repo-local retry contract docs resolved
  from `state.json.roadmap_dir`
- create round branches and round worktrees
- create worker branches and worker worktrees when `worker-plan.json` requires
  them
- create roadmap-update branches and worktrees after successful round merges
- update `orchestrator/state.json`, including active roadmap metadata when a
  reviewed and approved `update-roadmap` stage lawfully activates a new revision
- record artifact paths, retry-state fields, pending-merge stage, worker state
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
- roadmap update review
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
- Prefer reusing or resuming an existing compatible subagent before spawning a
  fresh one. A compatible subagent has the same runtime role, same round id or
  roadmap-update id, same branch/worktree, same selected lineage, and no
  unresolved instruction conflict with the current stage.
- Same-round retry should first go back to the prior compatible role subagent:
  planner retries to the planner, whole-round implementation retries to the
  implementer, integration retries to the integration implementer, worker
  retries to the named worker, review-only retries to the reviewer, and merge
  note retries to the merger.
- Resume a closed but resumable compatible subagent when the host supports it;
  otherwise send the next task to an idle compatible subagent. If no compatible
  prior subagent exists, if the prior subagent is non-observable, stale,
  role-mismatched, or unavailable, use a fresh subagent.
- Do not reuse a subagent across role boundaries. In particular, a reviewer may
  not implement fixes from its own review, an implementer may not review its
  own work, and a recovery investigator may not become the substantive stage
  author or reviewer.
- The `recovery-investigator` runtime role is
  `orchestrator/roles/recovery-investigator.md`.
- The `recovery-investigator` may not act as the substantive stage reviewer.
- Never interrupt a live subagent.
- Never set a cancellation timeout on a live subagent. Non-interrupting
  observation intervals are allowed for liveness checks.
- Wait for the subagent to finish before continuing.
- If a live subagent becomes non-observable, do not hang forever and do not
  interrupt it. Re-read controller-visible artifacts, check any host-provided
  non-interrupting status surface, and enter the recovery ladder when three
  consecutive observations show no status change, no artifact progress, and no
  trustworthy liveness signal.
- Do not convert a failed role-stage launch directly into terminal blockage
  unless `recovery-investigator` has been attempted or deterministically ruled
  out, and no same-round recovery, re-observation, or different-mechanism
  re-dispatch remains lawful.
- If a required `orchestrator/roles/<role>.md` file is missing, stop and record
  the exact controller error instead of inventing one.

## The Orchestrator May Not Do

- author `selection.md`, `plan.md`, `worker-plan.json`, `implementation-notes.md`,
  `review.md`, `review-record.json`, `merge.md`, `roadmap-update.md`, or
  `roadmap-update-review.md`
- choose worker ownership boundaries on its own
- perform substantive code integration or refresh itself
- approve work without a round-level reviewer

## If Real Subagents Are Unavailable

Record the precise blockage in `orchestrator/state.json`, then tell the user
the runtime skill cannot honor its delegation contract in the current
environment.
