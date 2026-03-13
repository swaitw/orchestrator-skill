# Worktree and Merge Rules

## Branches

- Use one branch per round.
- Prefix every round branch with `codex/`.
- Include the round id and a short task slug when practical.

Example:

- `codex/round-003-add-readme`

## Worktrees

- Use one dedicated worktree per round.
- Prefer `.worktrees/<round-id>` in the repository root.
- Ensure `.worktrees/` is gitignored before creating the worktree.
- Reuse the same worktree for planner, implementer, and reviewer within the round.

## Merge Rules

- Merge only after reviewer approval.
- Use squash merge into the recorded base branch.
- Keep the round branch until the squash merge succeeds.

## Round Finalization

After a successful squash merge:

- clear active round fields in `state.json`
- set `last_completed_round`
- advance to `update-roadmap`
- let the guider update `orchestrator/roadmap.md`
