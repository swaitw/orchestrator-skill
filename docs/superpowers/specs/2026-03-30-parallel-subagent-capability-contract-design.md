# Parallel Subagent Capability Contract Design

## Goal

Make the orchestrator runtime contract unambiguous about what "parallel"
means.

The contract must describe real concurrent delegated execution while remaining
agent-neutral. It must not depend on any one host's API shape, but it may
require that supported hosts provide concrete concurrent subagent capability.

## Problem

The repository already describes:

- parallel rounds
- worker fan-out inside a round
- controller state that can track multiple live rounds

But the runtime contract still contains single-active-subagent wording such as
"Wait for the active subagent to finish before continuing." That creates a
contradiction:

- the public model says parallel execution is supported
- the runtime wording still permits serialized execution

As written, the contract is parallel-capable in schema and design intent, but
not explicit enough about the execution requirement.

## Decision

The runtime contract will define parallel execution as a required host
capability, not an optional optimization.

The agent-neutral contract will require that a supported host can:

- launch multiple live delegated subagents concurrently
- preserve a stable handle for each live subagent
- wait for one or many specific live subagents without cancelling others
- resume controller work after individual subagent completion

If a host cannot provide those capabilities, it is unsupported for any runtime
state that requires concurrent execution.

## Compatibility Rule

This change does not make every non-concurrent host universally unsupported.

A host that lacks concurrent subagent support may still run a repository in a
strictly serial situation where the controller never needs more than one live
subagent at once.

The runtime must fail fast only when the active repo-local contract requires
concurrency that the host cannot honor. Examples:

- `max_parallel_rounds` greater than `1` and the scheduler has multiple
  dispatchable rounds
- `active_rounds` already contains multiple live rounds
- a round's `worker-plan.json` requires parallel worker execution
- repo-local recovery requires keeping one delegated activity live while
  launching another

The runtime must not silently degrade these cases to serial execution.

## Required Host Capabilities

The contract is intentionally agent-neutral, so it defines capabilities rather
than tool names.

Every supported host must provide:

1. **Concurrent launch**
   Start two or more delegated agents without blocking the controller on the
   first launch.
2. **Stable identity**
   Return a stable agent handle that can be persisted in controller state when
   needed for resume and reconciliation.
3. **Independent waiting**
   Allow the controller to wait on one specific live agent or a selected set of
   live agents without interrupting unrelated live agents.
4. **Completion inspection**
   Return final status and role output for each completed agent.
5. **Non-destructive background progress**
   Keep live agents running while the controller performs bookkeeping or waits
   on other agents.

Hosts may expose these capabilities however they want. The runtime contract
cares only that the capability exists and behaves deterministically enough for
controller state to remain correct.

## Runtime Semantics

The runtime contract must stop describing a single "active subagent."

Instead:

- each delegated round stage owns its own live delegated agent, if currently
  running
- each worker slice in worker fan-out owns its own live delegated agent, if
  currently running
- the controller may have multiple live delegated agents at once
- the controller waits only on the agent or agent set relevant to the next
  legal state transition

The controller still remains conservative:

- it never skips round stage order
- it never invents parallel work not authorized by repo-local artifacts
- it never interrupts a live delegated agent unless the user explicitly
  interrupts

## Round Parallelism

Parallel rounds remain opt-in and repo-local.

The controller may run multiple rounds concurrently only when:

- roadmap metadata marks the items parallel-safe
- dependency and merge-order rules permit concurrent progress
- `max_parallel_rounds` allows it
- the host supports concurrent delegated agents

When these conditions hold, the controller must dispatch separate delegated
agents for eligible rounds and track them independently.

## Worker Fan-Out

Worker fan-out remains planner-authored and opt-in.

When `worker-plan.json` defines multiple safe worker slices, the controller
must treat those slices as genuinely concurrent delegated work, not as a hint
to run workers one by one.

If the host cannot keep those workers live concurrently, the runtime must
record a precise controller blockage and stop instead of serializing the
worker plan.

## Resume and Reconciliation

The runtime contract must describe how concurrent live delegation resumes.

The controller must persist enough machine state to answer:

- which delegated agents are still expected to be live
- which round or worker each live agent belongs to
- which stage each live agent owns
- whether a missing live agent is recoverable, completed, or blocked

Resume rules must remain controller-visible. The controller must not infer
hidden host state from chat history.

## Failure Model

Unsupported concurrent execution is a controller-level blockage, not a silent
fallback.

When concurrency is required but unsupported, the runtime must:

1. record the precise host-capability mismatch in `orchestrator/state.json`
2. preserve controller state without fabricating progress
3. stop and tell the user the runtime cannot honor the active contract on the
   current host

This is different from a normal delegated-stage failure. A host-capability
mismatch is deterministic, so the runtime must not pretend the issue is a
recoverable single-agent stage failure.

## Documentation Changes

The following documents must be aligned to the same rule set:

- `skills/run-orchestrator-loop/SKILL.md`
- `skills/run-orchestrator-loop/references/delegation-boundaries.md`
- `skills/run-orchestrator-loop/references/resume-rules.md`
- `README.md`
- `docs/superpowers/specs/2026-03-27-parallel-orchestrator-design.md`

Required wording changes:

- remove any phrase that implies only one active subagent can exist
- replace "wait for the active subagent" with per-round or per-worker waiting
  semantics
- state that parallel execution requires host support for concurrent live
  delegated agents
- state that unsupported hosts fail fast when concurrency is required

## Non-Goals

This change does not:

- invent a repo-local process runner
- define one universal host API for subagents
- relax the rule that parallelism must be explicitly authorized by repo-local
  artifacts
- permit silent fallback from parallel contracts to serialized execution

## Acceptance Criteria

- No runtime contract document refers to a single active subagent.
- The contract defines host concurrency as an explicit required capability.
- The contract says unsupported hosts fail fast when concurrency is required.
- Round parallelism and worker fan-out are both described as real concurrent
  delegated execution.
- README, runtime docs, and the parallel design spec describe the same
  semantics.
