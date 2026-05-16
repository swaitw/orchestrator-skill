# Basic Serial Workflow

This file is the default runtime Interface for the common one-round-at-a-time
path. Use it first when `max_parallel_rounds` is `1`, no active
`state.json.roadmap_update` exists, no live round is `blocked`, no unresolved
resume errors exist, and no `round-plan-record.json` has authorized worker
fan-out.

Load advanced references only when one of the exits below triggers.

## Default Startup

1. Read `orchestrator/state.json`.
2. Load `orchestrator/artifact-manifest.md`.
3. Load `orchestrator/active-roadmap-bundle.md`.
4. Resolve `roadmap-view.json` and `verification.md` from `state.json.roadmap_dir`.
5. If `roadmap_update` is not `null`, exit to semantic roadmap update.
6. If `controller_stage` is `blocked`, a live round is `blocked`, or any
   resume error exists, exit to recovery.
7. If no live rounds exist and the active roadmap bundle is terminal, stop.
8. If no live rounds exist and unfinished milestones remain, create one round
   branch/worktree, record it in `active_rounds[]` at stage `plan`, and
   dispatch the planner.
9. If exactly one live serial round exists, resume that round from its recorded
   stage.
10. Otherwise exit to the relevant advanced Module.

## Default Round Path

Use one canonical branch/worktree under
[worktree-finalization-rules.md](worktree-finalization-rules.md). The newly
dispatched round starts at `plan`; `selection-record.json` is not expected
until the planner writes it.

Follow the serial legal stage order in [state-machine.md](state-machine.md):
`plan` -> `implement` -> `review` -> `finalize-round` -> `done`, dispatching
the matching repo-local role prompt for delegated stages. Re-read `state.json`
and the active roadmap bundle after finalization, then start another serial
round if unfinished milestones remain or enter terminal `done`.

Rejected reviews do not create a new round and do not enter finalization.
Consume `review-record.json.retry_target` from the canonical round worktree:
`implement` redispatches the implementer for bounded fixes, `plan`
redispatches the planner for a same-round replan, and `blocked` enters
recovery under [resume-rules.md](resume-rules.md). Keep the same round id,
branch, and worktree for every same-round retry.

## Advanced Exits

Exit this basic path and load the named Module when:

- `active_rounds` contains more than one round or `max_parallel_rounds > 1`:
  load `state-machine.md` and `resume-rules.md`.
- `round-plan-record.json.worker_mode` is `fanout`: load
  `orchestrator/round-plan-record-schema.md` and the worker fan-out section of
  `resume-rules.md`.
- `review-record.json.roadmap_closeout.mode` is
  `semantic-update-required`: load `orchestrator/roadmap-update-schema.md`.
- A round reaches `finalize-round`: load `worktree-finalization-rules.md` when
  merge admissibility, base freshness, or semantic update serialization needs
  the full predicate set.
- Any stage becomes non-observable, blocked, stale, or inconsistent: load
  `resume-rules.md` and `delegation-boundaries.md`.
- Required state or contract files are missing or malformed: load
  `resume-rules.md` corrupt-state and migration-needed rules.

The basic path is a front door, not a shortcut around the contracts. When an
advanced exit triggers, the advanced Module becomes authoritative for that
decision.
