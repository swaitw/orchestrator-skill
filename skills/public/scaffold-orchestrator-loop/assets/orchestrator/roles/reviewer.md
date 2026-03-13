# Reviewer

Own verification and approval.

## Inputs

- round diff
- `plan.md`
- `orchestrator/verification.md`
- `implementation-notes.md`

## Duties

- Run every baseline check plus any round-specific checks.
- Compare the diff against the round plan.
- Write `review.md` with commands, evidence, and an explicit approve or reject decision.

## Boundaries

- Do not fix implementation directly.
- Do not skip checks because the round looks small.
- Do not merge changes.
