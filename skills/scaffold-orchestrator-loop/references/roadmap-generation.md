# Roadmap Generation

Build a fresh active roadmap family from the approved alignment plus a fast
repository survey.

## Inputs

- User goal
- Approved alignment strategy from
  [alignment-brainstorm.md](alignment-brainstorm.md)
- Existing code, docs, tests, and config
- Current branch status
- Constraints discovered in the repo
- For `next-family`, `orchestrator/state.json`
- For `next-family`, the active roadmap bundle resolved from `roadmap_dir`
- For `next-family`, existing family ids under `orchestrator/roadmaps/`
- Existing `orchestrator/project-contract.md` when present

## Process

1. Survey the repository before drafting tasks.
2. Complete the alignment brainstorm and use the approved strategy as the
   roadmap source of truth.
3. If `orchestrator/` already exists, confirm the prior family is finished,
   detect `contract_version`, load
   `orchestrator/active-roadmap-bundle.md`, and record any constraints inherited
   from the live control plane. If that file is missing, stop as
   migration-needed before drafting a new family.
4. Choose a stable descriptive slug that names the control-plane family rather
   than one transient round.
5. Mint `roadmap_id` as `YYYY-MM-DD-NN-<slug>`, where `YYYY-MM-DD` is the
   scaffold date in the repo-local controller context and `NN` is that day's
   next unused zero-based roadmap-family ordinal (`00`, `01`, ...).
6. Initialize the fresh family at `rev-001`.
7. Identify the smallest meaningful milestones that move the approved strategy
   forward.
8. Sequence milestones so each one leaves the repo in a coherent state.
9. Record dependencies only when they change ordering.
10. Keep later milestones coarse while still giving the guider enough direction
   to extract concrete round work.

## Roadmap Rules

- Follow `orchestrator/active-roadmap-bundle.md` for the required `roadmap.md`
  sections, `roadmap-view.json` machine view, required milestone fields,
  required candidate-direction fields, terminal detection, and validation-error
  behavior.
- Record roadmap-specific retry policy, when needed, in `verification.md`
  `## Roadmap Overrides`; do not create a separate required retry-policy file.
- Make each milestone a coordination unit that is larger than a round and
  shaped around one delivery front or shared objective.
- Default milestones and directions to serial execution unless the repo survey
  gives you strong, explicit evidence that multiple delivery fronts are safe to
  advance together.
- Keep the roadmap strategic. Do not pre-write step-by-step implementation
  plans into the roadmap.
- Persist the approved alignment in `Alignment Summary`: thesis, success
  criteria, non-goals, chosen strategy, and deferred alternatives.
- Point to `orchestrator/project-contract.md` for repo-wide invariants instead
  of copying stable event schema, golden log, dry-run output, or package
  boundary rules into every roadmap revision.
- Prefer 3-7 initial milestones unless the goal is truly smaller.
- Write the new active roadmap to `orchestrator/roadmaps/<roadmap_id>/rev-001/roadmap.md`.
- Write the matching machine view to
  `orchestrator/roadmaps/<roadmap_id>/rev-001/roadmap-view.json`.
- Keep the descriptive slug stable once chosen; later revisions under the same roadmap family keep the same `roadmap_id`.
- Never reopen a completed family by appending items or reusing `rev-001`; mint
  a fresh family id instead.
- Follow `orchestrator/active-roadmap-bundle.md` for status-only round closeout
  versus semantic updates that must create `rev-00N+1` under the same
  `roadmap_id`.

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

- explicit connection to the approved alignment
- clear outcome
- obvious completion signal
- bounded extraction boundaries

## Revision Rules

The roadmap is expected to record accepted rounds. Keep `roadmap.md` stable
enough to guide execution and keep `roadmap-view.json` small enough that the
controller can apply reviewer-approved status-only closeout without parsing
human prose.
Publish a new revision when required by
`orchestrator/active-roadmap-bundle.md`. Used older families and revisions
remain durable history.

Active revisions should describe live and future coordination only. Do not copy
all completed milestones, directions, or extracted items forward into every new
revision. Move completed detail into
`orchestrator/roadmaps/<roadmap_id>/roadmap-history.md`, or keep only compact
completion pointers in the active revision when the pointer does not change
sequencing for remaining work.
