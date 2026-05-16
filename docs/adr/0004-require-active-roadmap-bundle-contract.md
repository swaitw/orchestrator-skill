# ADR-0004: Require a repo-local active roadmap bundle contract

## Status

Accepted, amended by ADR-0007

## Context

After ADR-0001 removed the legacy roadmap style and ADR-0003 removed
top-level roadmap pointer stubs, the active roadmap bundle still had rules
spread across runtime resume rules, scaffold roadmap-generation rules, role
prompts, and verification guidance.

The repeated rules covered:

- required active bundle files;
- `roadmap.md` section shape and `roadmap-view.json` machine view;
- terminal detection;
- validation-error behavior;
- revision immutability;
- status-only closeout handling; and
- the relationship between `roadmap.md`, `verification.md`, roadmap-specific
  retry policy, and `roadmap-history.md`.

That made the Interface shallow: every caller needed to know nearly the same
roadmap details, and any change to the Strategy-backlog roadmap contract risked
drift across several files.

## Decision

Add `orchestrator/active-roadmap-bundle.md` as a required repo-local contract
file in scaffolded control planes.

The file defines the Interface for the active roadmap bundle:

- how `state.json.roadmap_dir` resolves the active revision;
- which active bundle files are required;
- how `roadmap-view.json` milestone statuses determine unfinished work;
- which malformed roadmap-view shapes are validation errors;
- how `verification.md` and roadmap-specific retry policy relate to the active
  revision;
- when status-only round closeout may stay in the same revision; and
- when a roadmap update must publish a new revision.

Runtime must load this contract before interpreting the active roadmap bundle.
If an existing control plane is missing it, runtime records a precise
migration-needed controller error in `state.json.resume_errors.controller` and
stops instead of falling back to scattered rules.

This is not a return to top-level roadmap pointer stubs. The active roadmap
content still lives only under
`orchestrator/roadmaps/<roadmap_id>/<roadmap_revision>/`. The new file describes
the contract for interpreting that bundle; it is not a shortcut copy of
`roadmap.md` or `verification.md`.

## Consequences

**Positive:**
- Active bundle semantics have one repo-local home.
- Runtime and scaffold docs can point to one Interface instead of repeating
  parsing and terminal-detection rules.
- Existing controls with older scattered rules fail clearly as migration-needed
  instead of silently using an outdated contract.

**Negative:**
- Existing orchestrator control planes need migration before the updated runtime
  can resume them.
- The scaffolded contract gains one required file.

**Neutral:**
- `state.json.roadmap_dir` remains the only pointer to the active revision.
- Top-level roadmap pointer stubs remain unsupported.
- ADR-0007 later removed the separate required active-bundle retry file;
  roadmap retry overrides now live in `verification.md` `## Roadmap Overrides`.
