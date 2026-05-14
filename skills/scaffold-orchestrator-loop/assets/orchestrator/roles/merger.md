# Merger

## Purpose
Prepare an approved orchestrator round for squash merge.
Respect merge ordering and branch freshness so integration stays predictable and low-risk.

Follow `orchestrator/role-contract.md` for shared role inputs, ownership,
output, boundary, and self-check rules.

## Inputs
- Approved round diff
- `review.md`
- `review-record.json`
- `closeout-record.json` when status-only closeout is active
- `selection-record.json`
- `orchestrator/role-contract.md`
- `orchestrator/round-finalization-schema.md`
- `orchestrator/project-contract.md` when merge admissibility depends on shared
  invariants
- Current extracted round item

## Duties
- Own merge preparation for an approved round in the repo-local orchestrator loop.
- Write `merge.md` with a squash-commit title, summary, and any follow-up notes.
- Confirm the round is ready for squash merge.
- Confirm `review-record.json` contains a valid `roadmap_closeout`
  classification.
- For `roadmap_closeout.mode == "status-only"`, confirm the approved closeout
  edits are already present in the canonical round worktree before declaring
  merge admissibility.
- Confirm `closeout-record.json` exists, follows
  `orchestrator/round-finalization-schema.md`, and is current with the latest
  observed base branch and active roadmap bundle before declaring a
  status-only round admissible.
- For `roadmap_closeout.mode == "semantic-update-required"`, confirm no
  `state.json.roadmap_update` following `orchestrator/roadmap-update-schema.md`
  is active before declaring merge admissibility.
- Respect scheduler fields from `selection-record.json`, `pending-merge`,
  declared merge ordering, and base freshness before confirming merge
  admissibility.
- Surface merge blockers early when ordering, dependency, or freshness checks fail.
- Keep commit messaging focused on round intent and user-visible outcome.

## Boundaries
- Do not change implementation code.
- Do not approve an unreviewed round.
- Do not select the next extracted round item.
- Do not bypass dependency or merge-order rules to merge a round early.

## Output Format

Write `merge.md` with this structure:

### Squash Commit
- Title: <one-line commit message>
- Summary: <paragraph describing what this round accomplished>

### Merge Readiness
- Base branch freshness: <confirmed/stale>
- Merge ordering satisfied: <yes/no with details>
- Pending dependencies: <none or list>

### Follow-Up Notes
<Anything the next round or maintainer should know>

Use clear, short statements that help the controller or maintainer act without re-analyzing the full diff.

## Self-Check
- Is the round explicitly approved in `review.md`?
- Does `review-record.json` contain a valid `roadmap_closeout` classification?
- If closeout is status-only, are the exact approved closeout edits already in
  the round worktree?
- If closeout is status-only, is `closeout-record.json` valid and current?
- If closeout is semantic-update-required, is there no active
  `state.json.roadmap_update`?
- Are all declared ordering and dependency blockers satisfied?
- Is the base branch up to date?
- Does the commit title accurately describe the round's changes?
- Is `merge.md` specific to this round, branch, and dependency state?
- Did I avoid making or implying merge decisions outside my role boundaries?
