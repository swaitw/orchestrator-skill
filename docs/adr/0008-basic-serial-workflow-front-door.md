# ADR-0008: Add a basic serial workflow front door

## Status

Accepted, partially superseded by ADR-0010

ADR-0010 folds round selection into the planner-owned `plan` stage.

## Context

ADR-0004 through ADR-0007 moved duplicated workflow rules into deeper
repo-local contracts. That improved Locality, but it also made the runtime
entrypoint feel too heavy: a controller had to reason about recovery,
parallelism, worker fan-out, status-only closeout, semantic roadmap updates,
merge ordering, and all schema files before seeing the normal one-round path.

Most repositories start with `max_parallel_rounds: 1`, no active semantic
roadmap update, no worker fan-out, and no recovery state. For that common case,
the full contract is more Interface than the caller needs up front.

## Decision

Add `run-orchestrator-loop/references/basic-serial-workflow.md` as the default
runtime Interface.

The basic serial workflow covers only:

- startup checks needed for the normal path;
- one canonical round branch and worktree;
- `plan` -> `implement` -> `review` -> `finalize-round`;
- terminal roadmap recheck; and
- explicit advanced exits.

Advanced behavior remains in its existing Modules and is loaded only when a
trigger appears:

- parallel or non-serial live rounds, `blocked`, or `resume_errors` load
  `resume-rules.md`;
- worker fan-out loads `round-plan-record-schema.md` and worker fan-out resume
  rules;
- semantic roadmap updates load `roadmap-update-schema.md`;
- finalization predicates load `worktree-finalization-rules.md`; and
- subagent/recovery behavior loads `delegation-boundaries.md`.

## Consequences

**Positive:**
- The default runtime path has a smaller Interface.
- Safety contracts stay intact for advanced paths.
- New operators can understand the normal loop before reading recovery and
  fan-out machinery.

**Negative:**
- Runtime docs now have one more reference file.
- The front door must stay aligned with state-machine and resume rules.

**Neutral:**
- This does not remove any advanced behavior.
- `state.json` remains the machine authority for live controller state.
