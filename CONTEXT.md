# Context

## Glossary

- **Orchestrator control plane**: The repo-local `orchestrator/` directory that stores machine state, roadmap bundles, role prompts, round artifacts, and worktree paths for delegated execution.
- **Strategy-backlog roadmap**: The only supported roadmap shape for current scaffolds. It organizes work into milestones, candidate directions, and extracted round items.
- **Round**: A bounded unit of delegated work selected from the active roadmap bundle and tracked in `state.json.active_rounds[]`.
- **Role subagent**: A delegated agent running one orchestrator role from `orchestrator/roles/`, such as guider, planner, implementer, reviewer, merger, or recovery investigator.
- **Compatible prior handle**: A previously launched role subagent that still matches the same role, round, branch, worktree, and lineage closely enough to resume instead of spawning a fresh subagent.
- **Legacy compatibility layer**: The removed `legacy-flat` roadmap and top-level mirror-field support described in `docs/adr/0001-drop-legacy-flat-and-mirror-fields.md`.
