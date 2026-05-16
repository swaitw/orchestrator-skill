# ADR-0011: Make review feedback retry targets explicit

## Status

Accepted

## Context

The workflow already had a delegated `review` stage and prose that allowed a
rejected round to retry. The machine contract was weaker than the prose:

- `state-machine.md` named `review -> plan` but not ordinary
  `review -> implement` fix feedback;
- `resume-rules.md` referred to the repo-local review contract without naming
  the record field that drives the retry; and
- `review-record.json` required approval closeout classification, but rejected
  reviews did not carry a machine-readable retry target.

That left controllers to infer the next stage from `review.md` prose or chat
history, which conflicts with the repo-local control-plane design.

## Decision

Rejected reviews must write a `review-record.json` with:

- `decision: "rejected"`;
- `retry_target: "implement"`, `"plan"`, or `"blocked"`; and
- non-empty `required_changes`.

Use `retry_target: "implement"` for feedback that can be fixed inside the
existing round plan. Use `retry_target: "plan"` when the rejection changes the
round plan, selected scope, verification strategy, or worker fan-out. Use
`retry_target: "blocked"` only when the reviewer finds no lawful same-round
retry target.

The controller consumes the retry target to keep the same round, branch, and
worktree, then redispatches the owning role. It does not infer retry targets
from prose and does not let the reviewer implement its own findings.

## Consequences

**Positive:**
- Review feedback now has an explicit update loop.
- Ordinary review findings can go directly back to implementation without a
  forced replan.
- Same-round retries stay resumable from repository artifacts instead of chat
  history.

**Negative:**
- Existing control planes with older rejected review records need recovery or
  migration before retry can be automated.

**Neutral:**
- Approval still requires `roadmap_closeout` classification before
  finalization.
- Semantic roadmap updates remain separate from ordinary review feedback.
