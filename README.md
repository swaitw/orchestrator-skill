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
- scaffolding revisioned roadmap bundle files (for example `orchestrator/roadmaps/<roadmap_id>/<roadmap_revision>/roadmap.md`, `.../verification.md`, and `.../retry-subloop.md`, which stays present even when same-round retry is disabled), `orchestrator/state.json`, repo-local role definitions under `orchestrator/roles/`, and `orchestrator/worktrees/`
- preparing the repository for per-round worktrees
- creating the initial checkpoint commit

### `run-orchestrator-loop`

Use this skill after setup, when the repository already has an initialized `orchestrator/` directory.

It is responsible for:

- loading persisted orchestration state
- resuming current live rounds or starting new ones up to the configured cap
- delegating `select-task`, `plan`, `implement`, `review`, and `merge` stages to role subagents that load their runtime instructions from `orchestrator/roles/`, reusing compatible prior handles when available
- updating only controller-owned state
- squash-merging approved rounds, handling `pending-merge`, and advancing the
  roadmap through a reviewable `update-roadmap` artifact

## Workflow Model

The runtime loop is strict about ownership:

- The orchestrator is a controller, not an implementer.
- Serial execution is the default unless roadmap lanes, candidate-direction
  hints, and extracted-scope boundaries explicitly allow concurrency.
- Each live round uses one canonical branch plus one canonical worktree.
- Different rounds may be active at the same time when the roadmap and controller state authorize it.
- Review rejection or drift can send a round back to `plan`, `implement`, or `review` again.
- Worker fan-out is allowed only when the planner authors machine-readable `worker-plan.json` for a round.
- Roadmap updates after merges are delegated and reviewable before activation.
- Resume behavior comes from files in the repository, not hidden session context.

Per-round stage order is:

1. `select-task`
2. `plan`
3. `implement`
4. `review`
5. `pending-merge`
6. `merge`
7. `done`

Controller-global stage order is:

1. `dispatch-rounds`
2. `update-roadmap`
3. `done`

## Repo-Local Contract

The scaffolded repository gets a visible top-level `orchestrator/` directory with role definitions, rounds, roadmaps, and worktree staging kept in the repo:

```text
orchestrator/
├── state.json
├── state-schema.md
├── project-contract.md
├── worker-plan-schema.md
├── roles/
│   ├── guider.md
│   ├── planner.md
│   ├── implementer.md
│   ├── reviewer.md
│   ├── merger.md
│   └── recovery-investigator.md
├── rounds/
├── roadmap-updates/
├── roadmaps/
│   └── <roadmap_id>/
│       └── <roadmap_revision>/
│           ├── roadmap.md
│           ├── verification.md
│           └── retry-subloop.md
└── worktrees/
```

Key ideas behind that contract:

- `state.json` stays machine-oriented and tracks controller state,
  `active_rounds`, `roadmap_update`, and structured resume errors.
- Human-facing reasoning stays in the active roadmap bundle `orchestrator/roadmaps/<roadmap_id>/<roadmap_revision>/roadmap.md`, repo-local role definitions, and round artifacts.
- Roadmaps use milestones plus candidate directions; the guider extracts
  round-sized work from dependency-ready directions and may select concurrent
  slices when roadmap lanes and boundaries make that safe.
- Each round folder stores delegated artifacts such as `selection.md`, `plan.md`, `implementation-notes.md`, `review.md`, and `merge.md`.
- Round artifacts record extraction lineage so planner, reviewer, and merger
  work stays traceable back to the active roadmap bundle.
- Worker fan-out adds controller-readable `worker-plan.json` plus worker handoff artifacts, but approval and merge stay canonical at the round level.
- Roadmap updates after round merge are recorded under
  `orchestrator/roadmap-updates/` and require reviewer approval.
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
- `references/` contains supporting rules for roadmap generation, state transitions, resume behavior, and merge boundaries.
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
- Worker fan-out uses planner-authored `worker-plan.json` and worker worktrees beneath `orchestrator/worktrees/`.
- Reviewer approval is required before merge.
- Merge strategy is squash merge into the recorded base branch.
