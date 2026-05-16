# Orchestrator Skill Set

This repository contains a pair of agent-neutral skills for running a strict
repo-local orchestration workflow:

- `scaffold-orchestrator-loop` initializes an `orchestrator/` control plane in a target repository.
- `run-orchestrator-loop` resumes and coordinates the delegated round loop without doing the substantive work itself.

The design is intentionally explicit. State lives in the repository, delegated
role definitions are inspectable, and each round uses a dedicated branch and
worktree so progress can be resumed without relying on chat history. Serial
execution remains the default, but the repo-local contract can now opt into
parallel rounds and planner-defined worker fan-out explicitly.

## Included Skills

### `scaffold-orchestrator-loop`

Use this skill when a repository does not yet have an `orchestrator/` directory.

It is responsible for:

- surveying the repository and current goal
- running a deep alignment brainstorm before roadmap generation
- getting explicit approval of the roadmap strategy
- generating the first roadmap
- scaffolding `orchestrator/active-roadmap-bundle.md`, revisioned roadmap bundle
  files (for example
  `orchestrator/roadmaps/<roadmap_id>/<roadmap_revision>/roadmap.md`,
  `.../roadmap-view.json`, and `.../verification.md`), shared contracts such as
  `artifact-manifest.md`, `role-contract.md`, and schema files,
  `orchestrator/state.json`, repo-local role definitions under
  `orchestrator/roles/`, and `orchestrator/worktrees/`
- preparing the repository for per-round worktrees
- creating the initial checkpoint commit

### `run-orchestrator-loop`

Use this skill after setup, when the repository already has an initialized `orchestrator/` directory.

It is responsible for:

- starting from the basic serial workflow before loading advanced recovery,
  worker fan-out, or roadmap-update machinery
- loading persisted orchestration state
- resuming current live rounds or starting new ones up to the configured cap
- delegating `plan`, `implement`, and `review` stages to role subagents that
  load their runtime instructions from `orchestrator/roles/`; the planner owns
  normal task selection and writes `selection-record.json`
- updating only controller-owned state
- finalizing approved rounds by applying reviewer-approved status-only closeout
  when needed, deriving merge admissibility, squash-merging, and using a
  reviewable `update-roadmap` artifact only for semantic roadmap updates

## Workflow Model

The runtime loop is strict about ownership:

- The orchestrator is a controller, not an implementer.
- The basic serial workflow is the default path: one round branch, one
  worktree, `plan` -> `implement` -> `review` -> `finalize-round`, then a
  roadmap recheck.
- Serial execution is the default unless roadmap lanes, candidate-direction
  hints, and extracted-scope boundaries explicitly allow concurrency.
- Each live round uses one canonical branch plus one canonical worktree.
- Different rounds may be active at the same time when the roadmap and controller state authorize it.
- Review rejection or drift can send a round back to `plan`, `implement`, or `review` again.
- Worker fan-out is allowed only when the planner authors machine-readable
  worker assignments in `round-plan-record.json` for a round.
- Status-only round closeout is controller-owned before merge when
  `review-record.json` approves status selectors and compact completion
  pointers that resolve through `roadmap-view.json`.
- Semantic roadmap updates after round merges are delegated and reviewable before
  activation.
- Resume behavior comes from files in the repository, not hidden session context.
- Advanced references are loaded only when the basic serial workflow exits for
  recovery, worker fan-out, parallel rounds, or semantic roadmap update.

Per-round stage order is:

1. `plan`
2. `implement`
3. `review`
4. `finalize-round`
5. `done`

Controller-global stage order is:

1. `dispatch-rounds`
2. `update-roadmap`
3. `done`

Round finalization is controller-owned. It applies status-only closeout when
needed, derives merge admissibility, performs the squash merge, and enters
`update-roadmap` only after merge when the approved record requires a semantic
roadmap update.

## Repo-Local Contract

The scaffolded repository gets a visible top-level `orchestrator/` directory with role definitions, rounds, roadmaps, and worktree staging kept in the repo:

```text
orchestrator/
├── state.json
├── state-schema.md
├── artifact-manifest.md
├── project-contract.md
├── active-roadmap-bundle.md
├── role-contract.md
├── selection-record-schema.md
├── round-plan-record-schema.md
├── round-finalization-schema.md
├── roadmap-update-schema.md
├── roles/
│   ├── guider.md
│   ├── planner.md
│   ├── implementer.md
│   ├── reviewer.md
│   └── recovery-investigator.md
├── rounds/
├── roadmap-updates/
├── roadmaps/
│   └── <roadmap_id>/
│       ├── roadmap-history.md
│       └── <roadmap_revision>/
│           ├── roadmap.md
│           ├── roadmap-view.json
│           └── verification.md
└── worktrees/
```

Key ideas behind that contract:

- `state.json` stays machine-oriented and tracks controller state,
  `active_rounds`, `roadmap_update`, and structured resume errors.
- `artifact-manifest.md` owns the file layout, artifact keys, and path
  resolution rules used by runtime and roles.
- `role-contract.md` owns shared role obligations so role prompts stay focused
  on role-specific behavior.
- Human-facing reasoning stays in the active roadmap bundle
  `orchestrator/roadmaps/<roadmap_id>/<roadmap_revision>/roadmap.md`,
  repo-local role definitions, and round artifacts.
- The repo-local Active roadmap bundle Interface lives in
  `orchestrator/active-roadmap-bundle.md`; runtime treats an existing control
  plane missing that file as migration-needed instead of falling back to older
  scattered roadmap rules.
- Roadmaps use milestones plus candidate directions; the planner chooses the
  next dependency-ready round and may select lawful concurrent slices when
  roadmap lanes and boundaries make that safe.
- `roadmap-view.json` is the machine view for terminal detection, roadmap
  anchors, milestone ids, and direction ids; `roadmap.md` stays the human
  coordination narrative.
- Each round folder stores artifacts such as `selection-record.json`,
  `plan.md`, `implementation-notes.md`, `review.md`, and
  `review-record.json`; status-only rounds also store
  controller-authored `closeout-record.json`.
- `selection-record.json` records extraction lineage and scheduler fields so
  planner, reviewer, and finalization work stays traceable without duplicating
  those fields in `state.json`.
- `round-plan-record.json` records the planner's machine-readable round plan
  and optional worker fan-out assignments; selection, approval, and merge stay
  canonical at the round level.
- Status-only round closeout uses reviewer-approved `review-record.json`
  classification, runs before merge in the round worktree, and does not create
  a roadmap-update branch.
- Status-only closeout writes compact controller-owned `closeout-record.json`
  during `finalize-round` and revalidates it before merge after base refresh.
- Semantic roadmap updates after round merge are recorded under
  `orchestrator/roadmap-updates/`, follow
  `orchestrator/roadmap-update-schema.md`, and require reviewer approval.
- The runtime skill loads role instructions from `orchestrator/roles/*.md`.

## Repository Layout

This repository ships the skills under `skills/`:

```text
skills/
├── scaffold-orchestrator-loop/
│   ├── SKILL.md
│   ├── agents/openai.yaml
│   ├── assets/orchestrator/
│   │   ├── roles/
│   │   ├── rounds/
│   │   ├── roadmaps/
│   │   └── worktrees/
│   └── references/
└── run-orchestrator-loop/
    ├── SKILL.md
    ├── agents/openai.yaml
    └── references/
```

- `SKILL.md` is the entrypoint instruction file.
- `references/` contains supporting rules for roadmap generation, state
  transitions, resume behavior, and finalization boundaries.
- `run-orchestrator-loop/references/basic-serial-workflow.md` is the runtime
  front door for the common one-round-at-a-time path.
- `assets/` contains the starter `orchestrator/` state, role definitions, and worktree layout used by the scaffold skill.

## Installation

Use project-local installs as the documented supported path.

Run the commands below from the target repository root where these skills will
be used. Replace `<agent>` with your host agent identifier.

**Supported agents:** `claude-code`, `cursor`, `codex`, `gemini-cli`,
`github-copilot`, `opencode`

```bash
# install both skills
npx skills add https://github.com/soulomoon/orchestrator-skill --agent <agent> --skill scaffold-orchestrator-loop -y
npx skills add https://github.com/soulomoon/orchestrator-skill --agent <agent> --skill run-orchestrator-loop -y

# verify installation
npx skills ls --agent <agent>
```

Success means both `scaffold-orchestrator-loop` and `run-orchestrator-loop`
appear in the project skills list.

## Contributor Appendix

For local development, symlinks are useful when you want a checked-out repo to
act as the installed skill source. Keep this workflow as a contributor-only
shortcut, not the primary install path. Replace `<skill-dir>` with your host
agent's skill directory.

```bash
ln -s /path/to/orchestratorpattern/skills/scaffold-orchestrator-loop <skill-dir>/scaffold-orchestrator-loop
ln -s /path/to/orchestratorpattern/skills/run-orchestrator-loop <skill-dir>/run-orchestrator-loop
```

If either path already exists, move or remove the existing entry before creating
the symlink.

## Typical Usage

1. From the target repository root, install these skills into that repository for your agent host with `npx skills add ... --agent <id>`.
2. In a target repository, invoke `scaffold-orchestrator-loop` with the high-level goal.
3. Answer the alignment questions, compare the proposed roadmap strategies, and approve the direction.
4. Review the generated `orchestrator/` contract and initial checkpoint commit.
5. Invoke `run-orchestrator-loop` to start or resume the delegated round loop.
6. Let the runtime skill continue until roadmap milestones are complete or a recorded controller error blocks progress.
7. If the roadmap exposes independent lanes or clearly separable candidate directions, expect multiple round branches and worktrees to run concurrently.

## Development Notes

- Round branches are expected to use the `orchestrator/round-*` prefix.
- Round worktrees live under `orchestrator/worktrees/` in the target repository.
- Worker fan-out uses planner-authored `round-plan-record.json` and worker
  worktrees beneath `orchestrator/worktrees/`.
- Reviewer approval is required before merge.
- Merge strategy is squash merge into the recorded base branch.
