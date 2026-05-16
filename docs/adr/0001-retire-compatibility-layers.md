# ADR-0001: Retire compatibility layers

## Status

Accepted, amended by ADR-0007

## Context

`state.json` carried two kinds of legacy compatibility:

1. **Legacy mirror fields** — top-level aliases for round stage, task, branch,
   worktree, and artifact paths that duplicated data already present in
   `active_rounds[]` round records. These existed so older runtime readers that
   only understood single-round serial execution could read state from the top
   level.

2. **Legacy roadmap style** — an item-list roadmap shape predating the
   `strategy-backlog` milestone/direction/extraction contract. Every file that
   touched the roadmap had to dispatch on a style field and maintain separate
   parsing, terminal detection, and extraction rules for both styles.

Together these added:
- 6 redundant fields in the state schema
- Normalization logic in resume-rules.md (step 5 of startup)
- Mirror clearing rules in repo-contract.md and the worktree finalization rules
- Dual parsing paths in every style-dispatching rule
- A compatibility bridge across round records, review records, and
  verification artifacts
- A full legacy roadmap asset template and its associated contract

The interface was nearly as complex as the implementation — a hallmark of a
shallow module. Adding a third roadmap style would have required editing 13+
files.

## Decision

Remove both legacy compatibility layers entirely:

- Delete the 6 legacy mirror fields from the state schema and the state.json
  template.
- Delete the legacy roadmap style, its asset template, and all
  associated parsing/terminal/extraction rules.
- `strategy-backlog` is the only supported roadmap style.
- Round records in `active_rounds[]` are the single source of truth for round
  state.
- The compatibility bridge is removed from round records, review records, and
  verification artifacts.

Existing repos that still have legacy roadmaps or legacy mirror fields in
their `state.json` will need to migrate before running the updated runtime. The
migrations must update those control planes before running the current runtime.

## Consequences

**Positive:**
- State schema drops from 22 top-level fields to 16.
- Resume rules lose the normalization step — startup is simpler.
- Every style-dispatching rule becomes single-branch.
- The legacy roadmap asset and its contract are gone.
- Adding a new roadmap style in the future requires touching far fewer files.

**Negative:**
- Existing repos with legacy roadmaps cannot resume without migration.
- Older runtime readers that relied on top-level mirror fields will see missing
  fields or require migration.

**Neutral:**
- This ADR originally kept a roadmap style discriminator as cheap insurance.
  ADR-0007 later removed it because Strategy-backlog remained the only adapter and
  `contract_version` plus `orchestrator/active-roadmap-bundle.md` carry the
  compatibility signal.
