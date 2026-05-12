# Context

## Glossary

- **Orchestrator control plane**: The repo-local `orchestrator/` directory that stores machine state, roadmap bundles, role prompts, round artifacts, and worktree paths for delegated execution.
- **Strategy-backlog roadmap**: The only supported roadmap shape for current scaffolds. It organizes work into milestones, candidate directions, and extracted round items.
- **Round**: A bounded unit of delegated work selected from the active roadmap bundle. Live rounds are tracked only in `state.json.active_rounds[]`; resume preference is derived from that list, not from a separate pointer.
- **Pending-merge round**: A live round whose own `stage` is `pending-merge`. The control plane derives pending-merge queues from round records instead of persisting a separate top-level index.
- **Controller state shape**: The minimal persisted `state.json` interface the orchestrator control plane treats as authoritative. It should avoid fields that can be derived from canonical round or roadmap-update records.
- **Controller blockage**: A recoverable control-plane problem that is not owned by a specific round or roadmap-update record. It is recorded under structured controller error state, not as a separate scalar mirror.
- **Role subagent**: A delegated agent running one orchestrator role from `orchestrator/roles/`, such as guider, planner, implementer, reviewer, merger, or recovery investigator.
- **Compatible prior handle**: A previously launched role subagent that still matches the same role, round, branch, worktree, and lineage closely enough to resume instead of spawning a fresh subagent.
- **Legacy compatibility layer**: The removed `legacy-flat` roadmap and top-level mirror-field support described in `docs/adr/0001-drop-legacy-flat-and-mirror-fields.md`.
