# ADR-0001: Drop legacy-flat roadmap style and legacy mirror fields

## Status

Accepted

## Context

`state.json` carried two kinds of legacy compatibility:

1. **Legacy mirror fields** — 6 top-level fields (`stage`, `current_task`,
   `branch`, `worktree_path`, `active_round_dir`, `round_artifacts`) that
   duplicated data already present in `active_rounds[]` round records. These existed so older runtime readers that only understood
   single-round serial execution could read state from the top level.

2. **`legacy-flat` roadmap style** — a flat item-list roadmap shape predating
   the `strategy-backlog` milestone/direction/extraction contract. Every file
   that touched the roadmap had to dispatch on `roadmap_style` and maintain
   separate parsing, terminal detection, and extraction rules for both styles.

Together these added:
- 6 redundant fields in the state schema
- Normalization logic in resume-rules.md (step 5 of startup)
- Mirror clearing rules in repo-contract.md and worktree-merge-rules.md
- Dual parsing paths in every style-dispatching rule
- A `roadmap_item_id` compatibility bridge across round records, review records,
  and verification artifacts
- A full asset template (`legacy-flat-roadmap.md`) and its associated contract

The interface was nearly as complex as the implementation — a hallmark of a
shallow module. Adding a third roadmap style would have required editing 13+
files.

## Decision

Remove both legacy compatibility layers entirely:

- Delete the 6 legacy mirror fields from the state schema and the state.json
  template.
- Delete the `legacy-flat` roadmap style, its asset template, and all
  associated parsing/terminal/extraction rules.
- `strategy-backlog` is the only supported roadmap style.
- Round records in `active_rounds[]` are the single source of truth for round
  state.
- `roadmap_item_id` is removed from round records, review records, and
  verification artifacts.

Existing repos that still have legacy-flat roadmaps or legacy mirror fields in
their `state.json` will need to migrate before running the updated runtime. The
git history preserves the legacy-flat contract for reference during migration.

## Consequences

**Positive:**
- State schema drops from 22 top-level fields to 16.
- Resume rules lose the normalization step — startup is simpler.
- Every style-dispatching rule becomes single-branch.
- The `legacy-flat-roadmap.md` asset and its contract are gone.
- Adding a new roadmap style in the future requires touching far fewer files.

**Negative:**
- Existing repos with legacy-flat roadmaps cannot resume without migration.
- Older runtime readers that relied on top-level mirror fields will see missing
  fields or require migration.

**Neutral:**
- `contract_version` and `roadmap_style` fields remain in the schema.
  `roadmap_style` is now always `strategy-backlog`. These fields are cheap
  insurance if a future style variant is needed, and they still serve as a
  contract version marker.
