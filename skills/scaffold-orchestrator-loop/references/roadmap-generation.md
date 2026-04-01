# Roadmap Generation

Build a fresh active roadmap family from the goal plus a fast repository
survey.

## Inputs

- User goal
- Existing code, docs, tests, and config
- Current branch status
- Constraints discovered in the repo
- For `next-family`, `orchestrator/state.json`
- For `next-family`, the active roadmap bundle resolved from `roadmap_dir`
- For `next-family`, existing family ids under `orchestrator/roadmaps/`

## Process

1. Survey the repository before drafting tasks.
2. If `orchestrator/` already exists, confirm the prior family is finished and
   record any constraints inherited from the live control plane.
3. Choose a stable descriptive slug that names the control-plane family rather
   than one transient round.
4. Mint `roadmap_id` as `YYYY-MM-DD-NN-<slug>`, where `YYYY-MM-DD` is the
   scaffold date in the repo-local controller context and `NN` is that day's
   next unused zero-based roadmap-family ordinal (`00`, `01`, ...).
5. Initialize the fresh family at `rev-001`.
6. Identify the smallest meaningful milestones that move the goal forward.
7. Sequence milestones so each item leaves the repo in a coherent state.
8. Record dependencies only when they change ordering.
9. Keep later items coarse; make the next item concrete.

## Roadmap Rules

- Use an ordered list.
- Give each item one clear deliverable.
- Record status with `[pending]`, `[in-progress]`, or `[done]`.
- Include `Item id:`, `Depends on:`, `Parallel safe:`, `Parallel group:`, `Merge after:`, and `Completion notes:` lines for each item.
- Default items to serial execution unless the repo survey gives you strong, explicit evidence that two items are safe to run in parallel.
- Avoid speculative implementation detail far ahead even when the roadmap allows later parallel rounds.
- Prefer 3-7 initial items unless the goal is truly smaller.
- Write the new active roadmap to `orchestrator/roadmaps/<roadmap_id>/rev-001/roadmap.md`.
- Keep the descriptive slug stable once chosen; later revisions under the same roadmap family keep the same `roadmap_id`.
- Never reopen a completed family by appending items or reusing `rev-001`; mint
  a fresh family id instead.
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

The roadmap is expected to change after accepted rounds. Keep it stable enough
to guide execution, but lightweight enough that the guider can update it
without rewriting the whole file. Used older families and revisions remain
immutable history.
