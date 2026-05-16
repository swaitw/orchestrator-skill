# Worktree and Finalization Rules

Use [state-machine.md](state-machine.md) as the authoritative lifecycle table.
This file defines worktree conventions, finalization predicates, and squash
merge bookkeeping.

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
- Reuse the same canonical worktree for planner, implementer, reviewer,
  finalization, and any same-round retry attempts within the round.
- Use `orchestrator/artifact-manifest.md` for round artifact keys and
  live-vs-archived path resolution.
- Status-only round closeout is applied in this canonical round worktree during
  `finalize-round`, before squash merge, so the closeout edits are included in
  the round merge commit.

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
- Follow `orchestrator/roadmap-update-schema.md` for roadmap-update state,
  artifacts, review, rejection, and activation rules.
- Merge the roadmap-update branch only after explicit reviewer approval.
- Keep `state.json.roadmap_update` active until the update is approved, merged,
  and cleared under that schema.

## Finalize-Round

`finalize-round` is controller-owned. It performs all post-review work:

1. Read `selection-record.json`, `round-plan-record.json`, `review.md`, and the
   approved `review-record.json` from the canonical round worktree.
2. Verify reviewer approval and a valid `roadmap_closeout` classification.
3. If `roadmap_closeout.mode == "status-only"`, apply the approved closeout
   under `orchestrator/active-roadmap-bundle.md` and
   `orchestrator/round-finalization-schema.md`.
4. Derive merge admissibility from reviewer approval, valid finalization
   records, scheduler fields in `selection-record.json`, dependency state, base
   freshness, and active semantic roadmap-update state.
5. If admissibility is blocked, keep the round in `finalize-round` until the
   blocker clears or the state machine requires a step back to `plan`,
   `implement`, or `review`.
6. Squash merge the round into the recorded base branch.
7. Remove the round from `state.json.active_rounds`.
8. If the approved review requires semantic roadmap update, set
   `state.json.controller_stage` to `update-roadmap` and create the
   roadmap-update branch/worktree record. Otherwise re-read the active roadmap
   bundle and continue dispatch or terminal detection.

## Finalization Predicates

- Do not finalize a rejected round.
- Do not finalize an approved round unless `review-record.json` contains a
  valid `roadmap_closeout` classification.
- For `roadmap_closeout.mode == "status-only"`, do not merge until the exact
  approved closeout edits are present in the canonical round worktree and
  `closeout-record.json` validates against
  `orchestrator/round-finalization-schema.md`.
- For `roadmap_closeout.mode == "semantic-update-required"`, do not apply
  status-only roadmap edits in the round worktree; use the semantic
  `update-roadmap` path after merge.
- Do not merge a semantic-update-required round while
  `state.json.roadmap_update` is not `null`; keep the round in
  `finalize-round` until the active semantic update is approved, merged, and
  cleared.
- Revalidate closeout against the latest base branch and active roadmap bundle
  before merge when base or active bundle content changed after closeout.
- Do not merge a round ahead of declared extracted-item ordering or dependency
  rules from `selection-record.json`.

## Status-Only Closeout

When `review-record.json.roadmap_closeout.mode == "status-only"`:

- apply only the exact edits listed in `roadmap_closeout` to active roadmap
  bundle files in the canonical round worktree by resolving selectors through
  `roadmap-view.json`
- verify the resulting diff matches the approved closeout selectors
- write `closeout-record.json` following
  `orchestrator/round-finalization-schema.md`
- keep those edits on the round branch so the squash merge carries them into
  the base branch

If closeout revalidation fails, keep the round out of merge and step to
`review` or recovery according to the failure.
