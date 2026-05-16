# Context

## Glossary

- **Orchestrator control plane**: The repo-local `orchestrator/` directory that stores machine state, roadmap bundles, role prompts, round artifacts, and worktree paths for delegated execution.
- **Strategy-backlog roadmap**: The only supported roadmap shape for current scaffolds. It organizes work into milestones, candidate directions, and extracted round items.
- **Artifact manifest**: The repo-local `orchestrator/artifact-manifest.md` file that defines the canonical control-plane file layout, round artifact keys, worker artifact paths, roadmap-update artifact paths, and path-resolution rules.
- **Active roadmap bundle**: The revision directory named by `state.json.roadmap_dir`. It is the only source for live `roadmap.md`, `roadmap-view.json`, and `verification.md` content.
- **Active roadmap bundle contract**: The repo-local `orchestrator/active-roadmap-bundle.md` file that defines the Interface for reading the active roadmap bundle, validating `roadmap-view.json`, detecting unfinished milestones, applying status-only round closeout, and deciding when semantic roadmap updates must create a new revision.
- **Roadmap view**: The active revision's `roadmap-view.json` file. It is the machine-readable view for milestone ids, direction ids, dependencies, terminal detection, and status-only closeout anchors; `roadmap.md` remains the human coordination narrative.
- **Round**: A bounded unit of delegated work selected from the active roadmap bundle. Live rounds are tracked only in `state.json.active_rounds[]`; resume preference is derived from that list, not from a separate pointer.
- **Basic serial workflow**: The default runtime Interface for one-round-at-a-time execution when no advanced path is active. It covers startup, one round branch/worktree, `plan` -> `implement` -> `review` -> `finalize-round`, and terminal recheck before loading worker fan-out, semantic roadmap update, or recovery details.
- **Selection record**: The planner-authored `selection-record.json` artifact that stores selected round lineage and scheduler fields. It is the machine authority for selection data.
- **Round finalization**: The controller-owned post-review step that applies status-only closeout when needed, derives merge admissibility, performs squash merge bookkeeping, removes the live round, and dispatches semantic roadmap update when required.
- **Round finalization contract**: The repo-local `orchestrator/round-finalization-schema.md` file that defines the machine contract for reviewer-authored `review-record.json` and compact controller-authored `closeout-record.json`.
- **Status-only round closeout**: Controller-owned bookkeeping inside `finalize-round` that applies reviewer-approved milestone status selectors and compact completion pointers through `roadmap-view.json` anchors to the active roadmap revision copy in the canonical round worktree without changing future coordination meaning.
- **Semantic roadmap update**: A delegated, reviewable roadmap update that changes future coordination, milestone or direction meaning, sequencing, parallel lanes, extraction scope, verification meaning, or retry policy.
- **Roadmap update record**: The active `state.json.roadmap_update` object plus its paired `roadmap-update.md` and `roadmap-update-review.md` artifacts, governed by `orchestrator/roadmap-update-schema.md`.
- **Round plan record**: The planner-authored `round-plan-record.json` machine artifact that records the current round's plan lineage and optional worker fan-out assignments; `plan.md` remains the human plan.
- **Role contract**: The repo-local `orchestrator/role-contract.md` file that defines shared role inputs, ownership rules, output rules, boundaries, and self-checks so individual role prompts only carry role-specific behavior.
- **Retry policy**: Roadmap-specific retry overrides recorded in active bundle `verification.md` under `## Roadmap Overrides`; shared retry mechanics live in runtime references.
- **Controller state shape**: The minimal persisted `state.json` interface the orchestrator control plane treats as authoritative. It should avoid fields that can be derived from canonical round or roadmap-update records.
- **Controller blockage**: A recoverable control-plane problem that is not owned by a specific round or roadmap-update record. It is recorded under structured controller error state, not as a separate scalar mirror.
- **Role subagent**: A delegated agent running one orchestrator role from `orchestrator/roles/`, such as guider, planner, implementer, reviewer, or recovery investigator.
- **Compatible prior handle**: A previously launched role subagent that still matches the same role, round, branch, worktree, and lineage closely enough to resume instead of spawning a fresh subagent.
- **Legacy compatibility layer**: The removed roadmap and mirror-field compatibility support described in `docs/adr/0001-retire-compatibility-layers.md`.
