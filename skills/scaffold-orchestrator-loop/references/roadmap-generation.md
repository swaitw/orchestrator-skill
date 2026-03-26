# Roadmap Generation

Build the initial roadmap from the goal plus a fast repository survey.

## Inputs

- User goal
- Existing code, docs, tests, and config
- Current branch status
- Constraints discovered in the repo

## Process

1. Survey the repository before drafting tasks.
2. Choose a stable descriptive slug that names the control-plane family rather than one transient round.
3. Mint `roadmap_id` as `YYYY-MM-DD-NN-<slug>`, where `YYYY-MM-DD` is the scaffold date in the repo-local controller context and `NN` is that day's zero-based roadmap-family ordinal (`00`, `01`, ...).
4. Initialize the first revision as `rev-001`.
5. Identify the smallest meaningful milestones that move the goal forward.
6. Sequence milestones so each item leaves the repo in a coherent state.
7. Record dependencies only when they change ordering.
8. Keep later items coarse; make the next item concrete.

## Roadmap Rules

- Use an ordered list.
- Give each item one clear deliverable.
- Record status with `[pending]`, `[in-progress]`, or `[done]`.
- Include `Item id:`, `Depends on:`, `Parallel safe:`, `Parallel group:`, `Merge after:`, and `Completion notes:` lines for each item.
- Default items to serial execution unless the repo survey gives you strong, explicit evidence that two items are safe to run in parallel.
- Avoid speculative implementation detail far ahead even when the roadmap allows later parallel rounds.
- Prefer 3-7 initial items unless the goal is truly smaller.
- Write the initial roadmap to `orchestrator/roadmaps/<roadmap_id>/rev-001/roadmap.md`.
- Keep the descriptive slug stable once chosen; later revisions under the same roadmap family keep the same `roadmap_id`.
- Keep a roadmap revision immutable once any round uses it; later semantic updates should create `rev-00N+1` under the same `roadmap_id`.

Parallel metadata guidance:

- `Parallel safe: no` is the default.
- Use `Parallel safe: yes` only when the item's boundaries are clear and dependencies are already explicit.
- Use `Parallel group: none` for serial items.
- Use the same non-`none` `Parallel group:` only for items that may legally co-run.
- Use `Merge after:` to force merge ordering when dependencies alone are not enough.

## First Item Quality Bar

The first actionable roadmap item should be concrete enough that a guider can select it without guessing:

- clear outcome
- obvious verification target
- bounded scope

## Revision Rules

The roadmap is expected to change after accepted rounds. Keep it stable enough to guide execution, but lightweight enough that the guider can update it without rewriting the whole file.
