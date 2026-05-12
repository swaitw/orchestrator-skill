# ADR-0003: Remove top-level roadmap pointer stubs

## Status

Accepted

## Context

The orchestrator control plane stores the active roadmap family and revision in
`state.json` as `roadmap_id`, `roadmap_revision`, and `roadmap_dir`. Runtime
already resolves live roadmap files from `state.json.roadmap_dir`.

The scaffold contract still allowed optional top-level convenience pointer files:

- `orchestrator/roadmap.md`
- `orchestrator/verification.md`
- `orchestrator/retry-subloop.md`

Those stubs duplicated files already present inside the active roadmap bundle.
They were non-authoritative, but keeping them required setup, refresh, and review
rules to explain how they should match `state.json`.

## Decision

Remove support for top-level roadmap pointer stubs.

The active roadmap bundle under
`orchestrator/roadmaps/<roadmap_id>/<roadmap_revision>/` is the only source for:

- `roadmap.md`
- `verification.md`
- `retry-subloop.md`

Runtime and scaffold instructions must resolve these files from
`state.json.roadmap_dir`. New scaffolds and `next-family` setups do not create,
refresh, or validate top-level pointer stubs.

## Consequences

**Positive:**
- The control plane has one source for active roadmap content.
- Setup no longer has to refresh non-authoritative pointer files.
- Review checks no longer need to compare pointer stubs against `state.json`.

**Negative:**
- Users cannot open top-level shortcut files to find the active roadmap bundle.
  They must follow `state.json.roadmap_dir` or use repo-local tooling.
- Existing repos with old pointer stubs may keep them as historical files, but
  the updated contract ignores them.

**Neutral:**
- This does not change the active roadmap bundle layout or roadmap-update flow.
- `roadmap-history.md` remains under `orchestrator/roadmaps/<roadmap_id>/`.
