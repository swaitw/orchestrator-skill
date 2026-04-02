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
7. Sequence milestones so each one leaves the repo in a coherent state.
8. Record dependencies only when they change ordering.
9. Keep later milestones coarse while still giving the guider enough direction
   to extract concrete round work.

## Roadmap Rules

- Use a layered strategy-backlog document with `Goal`, `Outcome boundaries`,
  `Global sequencing rules`, `Parallel lanes`, and `Milestones`.
- Record milestone status with `[pending]`, `[in-progress]`, or `[done]`.
- Make each milestone a coordination unit that is larger than a round and
  shaped around one delivery front or shared objective.
- For each milestone, include `Milestone id:`, `Depends on:`, `Intent:`,
  `Completion signal:`, `Parallel lane:`, and `Coordination notes:`.
- Under each milestone, write `Candidate directions:` that describe plausible
  work fronts the guider may extract from.
- For each candidate direction, include `Direction id:`, `Summary:`, `Why it
  matters now:`, `Preconditions:`, `Parallel hints:`, `Boundary notes:`, and
  `Extraction notes:`.
- Default milestones and directions to serial execution unless the repo survey
  gives you strong, explicit evidence that multiple delivery fronts are safe to
  advance together.
- Keep the roadmap strategic. Do not pre-write step-by-step implementation
  plans into the roadmap.
- Prefer 3-7 initial milestones unless the goal is truly smaller.
- Write the new active roadmap to `orchestrator/roadmaps/<roadmap_id>/rev-001/roadmap.md`.
- Keep the descriptive slug stable once chosen; later revisions under the same roadmap family keep the same `roadmap_id`.
- Never reopen a completed family by appending items or reusing `rev-001`; mint
  a fresh family id instead.
- Keep a roadmap revision immutable once any round uses it; later semantic updates should create `rev-00N+1` under the same `roadmap_id`.

Parallel metadata guidance:

- Use `Parallel lanes` to advertise the obvious concurrent fronts the roadmap
  already knows about.
- Keep lane definitions stable enough that the guider can lawfully co-schedule
  extracted work across teams.
- Use `Parallel hints:` inside candidate directions to explain whether the
  guider should keep extraction serial, lane-bound, or broadly co-runnable.
- Do not force every legal concurrent combination into the roadmap. The guider
  may still assemble additional safe concurrent selections when boundaries and
  dependencies are explicit.

## Extraction Quality Bar

The first milestone and its candidate directions should be concrete enough that
a guider can extract lawful round work without guessing:

- clear outcome
- obvious completion signal
- bounded extraction boundaries

## Revision Rules

The roadmap is expected to change after accepted rounds. Keep it stable enough
to guide execution, but lightweight enough that the guider can update it
without rewriting the whole file. Publish a new revision when a refinement
changes future coordination, sequencing, boundaries, or shared interpretation.
Used older families and revisions remain immutable history.
