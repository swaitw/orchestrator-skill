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
- Status-only round closeout is applied in this canonical round worktree before
  squash merge, so the closeout edits are included in the round merge commit.
- The controller writes `orchestrator/rounds/<round-id>/closeout-record.json`
  in this worktree when status-only closeout is applied.

## Worker Branches and Worktrees

- Use worker branches only when planner-authored `round-plan-record.json`
  requires worker fan-out.
- Fork each worker branch from the canonical round branch baseline.
- Use one worker worktree per worker under
  `orchestrator/worktrees/<round-id>-<worker-id>`.
- Keep worker writes inside the assigned worker branch/worktree.
- Keep the canonical round branch/worktree reserved for integrated round state.

## Roadmap Update Branches And Worktrees

- Use a roadmap-update branch only when the approved `review-record.json`
  requires a semantic roadmap update.
- Follow `orchestrator/roadmap-update-schema.md` for the roadmap-update branch,
  worktree, update artifact, review artifact, rejection loop, and activation
  rules.
- Merge the roadmap-update branch only after explicit reviewer approval.
- Record the active roadmap-update branch, worktree, artifacts, revisions, and
  status in `state.json.roadmap_update` until the update is approved, merged,
  and cleared.

## Merge Rules

Use [state-machine.md](state-machine.md) as the authoritative lifecycle table.
This section defines merge predicates and git bookkeeping only.

- Merge only after reviewer approval that finalizes the round under the
  repo-local review contract.
- Do not merge a round unless the approved `review-record.json` contains a
  valid `roadmap_closeout` classification.
- Derive merge admissibility from reviewer approval, round finalization
  records, `selection-record.json` scheduler fields, dependency state, base
  freshness, and active semantic roadmap-update state. Do not rely on a
  persisted `merge_ready` boolean.
- For `roadmap_closeout.mode == "status-only"`, merge only after the controller
  has applied the exact approved closeout edits in the canonical round worktree
  and written a valid `closeout-record.json` proving those edits in that
  worktree.
- For `roadmap_closeout.mode == "semantic-update-required"`, do not apply
  status-only roadmap edits in the round worktree; use the semantic
  `update-roadmap` path after merge.
- Do not merge a semantic-update-required round while
  `state.json.roadmap_update` is not `null`; keep the round in `pending-merge`
  until the active semantic update is approved, merged, and cleared.
- A reviewed round may pause in `pending-merge` until merge ordering and base
  freshness requirements are satisfied.
- Before merging a status-only round that waited in `pending-merge`, revalidate
  `closeout-record.json` against the latest base branch and active roadmap
  bundle. If revalidation fails, return the round to `closeout`, `review`, or
  recovery instead of merging stale closeout edits.
- Use squash merge into the recorded base branch.
- Keep the round branch until the squash merge succeeds.
- Do not merge a round ahead of declared extracted-item ordering or dependency
  rules from `selection-record.json`.

## Status-Only Round Closeout

After reviewer approval and before squash merge:

- read the approved `review-record.json` from the canonical round worktree
- if `review-record.json` is missing, malformed, or lacks
  `roadmap_closeout.mode`, keep the round in `review` or recovery; do not merge
  and do not silently enter `update-roadmap`
- if `roadmap_closeout.mode` is `status-only`, set the round stage to
  `closeout`, apply only the exact edits listed in `roadmap_closeout` to the
  active roadmap bundle files in the canonical round worktree by resolving
  selectors through `roadmap-view.json`, and verify the resulting diff matches
  the approved closeout record
- if status-only closeout succeeds, write `closeout-record.json` following
  `orchestrator/round-finalization-schema.md`, keep those edits on the round
  branch, and proceed to `pending-merge` or `merge`
- if `roadmap_closeout.mode` is `semantic-update-required`, skip `closeout` and
  proceed to `pending-merge` or `merge`; the semantic roadmap update happens
  only after the round squash merge succeeds

## Round Finalization

After a successful squash merge:

- remove the merged round from `active_rounds`
- clear retry-state fields in `state.json` when the repo-local contract uses
  them
- read the merged round's approved `review-record.json`
- if `review-record.json` classifies closeout as status-only, verify the
  squash merge already contains the approved closeout edits and the matching
  `closeout-record.json`, leave `state.json.roadmap_update` as `null`, and
  stay in `dispatch-rounds` or move to `done` after re-reading the active
  roadmap bundle
- if `review-record.json` requires a semantic roadmap update, advance
  controller state to `update-roadmap` only when no other semantic roadmap
  update is active
- for semantic roadmap updates, dispatch the guider on the roadmap-update
  branch/worktree to author the next roadmap revision
- dispatch the reviewer to approve or reject the semantic roadmap update
  artifact and diff
- after approval and merge of the roadmap-update branch, update `state.json`
  `roadmap_id`, `roadmap_revision`, and `roadmap_dir` for the approved new
  revision, then clear `state.json.roadmap_update`
