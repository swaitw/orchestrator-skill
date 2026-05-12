# Worktree and Merge Rules

## Round Branches

- Use one canonical branch per round.
- Use the exact pattern `orchestrator/round-<nn>-<slug>`.
- Include the round number and a short task slug in every round branch.

Example:

- `orchestrator/round-03-add-readme`

## Round Worktrees

- Use one dedicated canonical worktree per round.
- Use `orchestrator/worktrees/<round-id>`.
- Ensure `orchestrator/worktrees/` is ignored by a tracked ignore rule before
  creating the worktree.
- Reuse the same canonical worktree for planner, integration implementer,
  reviewer, merger, and any same-round retry attempts within the round.
- While the round is live, `state.json` artifact paths are resolved inside this
  worktree. The parent checkout does not need to contain the latest round
  artifacts until the round branch merges.

## Worker Branches and Worktrees

- Use worker branches only when planner-authored `worker-plan.json` requires
  worker fan-out.
- Fork each worker branch from the canonical round branch baseline.
- Use one worker worktree per worker under
  `orchestrator/worktrees/<round-id>-<worker-id>`.
- Keep worker writes inside the assigned worker branch/worktree.
- Keep the canonical round branch/worktree reserved for integrated round state.

## Roadmap Update Branches And Worktrees

- After a successful round merge, use one roadmap-update branch:
  `orchestrator/roadmap-update-<round-id>-<slug>`.
- Use one roadmap-update worktree:
  `orchestrator/worktrees/roadmap-update-<round-id>`.
- The guider authors
  `orchestrator/roadmap-updates/<round-id>-roadmap-update.md` and any roadmap
  bundle edits in that worktree.
- The reviewer authors
  `orchestrator/roadmap-updates/<round-id>-roadmap-update-review.md`.
- Merge the roadmap-update branch only after explicit reviewer approval.
- Record the active roadmap-update branch, worktree, artifacts, revisions, and
  status in `state.json.roadmap_update` until the update is approved, merged,
  and cleared.

## Merge Rules

- Merge only after reviewer approval that finalizes the round under the
  repo-local review contract.
- A reviewed round may pause in `pending-merge` until merge ordering and base
  freshness requirements are satisfied.
- Use squash merge into the recorded base branch.
- Keep the round branch until the squash merge succeeds.
- Do not merge a round ahead of declared extracted-item ordering or dependency
  rules.

## Round Finalization

After a successful squash merge:

- remove the merged round from `active_rounds`
- remove the merged round id from `pending_merge_rounds`
- clear retry-state fields in `state.json` when the repo-local contract uses
  them
- set `last_completed_round`
- advance controller state to `update-roadmap`
- dispatch the guider on the roadmap-update branch/worktree to update milestone
  statuses in the active roadmap bundle or author the next roadmap revision
- dispatch the reviewer to approve or reject the roadmap update artifact and
  diff
- after approval and merge of the roadmap-update branch, update `state.json`
  `roadmap_id`, `roadmap_revision`, and `roadmap_dir` when the approved update
  activates a new revision
