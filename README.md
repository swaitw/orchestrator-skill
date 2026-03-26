# Orchestrator Skill Set

This repository contains a pair of Codex skills for running a strict repo-local orchestration workflow:

- `scaffold-orchestrator-loop` initializes an `orchestrator/` control plane in a target repository.
- `run-orchestrator-loop` resumes and coordinates the delegated round loop without doing the substantive work itself.

The design is intentionally explicit. State lives in the repository, delegated role definitions are inspectable, and each round uses a dedicated branch and worktree so progress can be resumed without relying on chat history.

## Included Skills

### `scaffold-orchestrator-loop`

Use this skill when a repository does not yet have an `orchestrator/` directory.

It is responsible for:

- surveying the repository and current goal
- generating the first roadmap
- scaffolding revisioned roadmap bundle files (for example `orchestrator/roadmaps/<roadmap_id>/<roadmap_revision>/roadmap.md`, `.../verification.md`, and `.../retry-subloop.md`, which stays present even when same-round retry is disabled), `orchestrator/state.json`, and repo-local role agents under `.codex/agents/`
- preparing the repository for per-round worktrees
- creating the initial checkpoint commit

### `run-orchestrator-loop`

Use this skill after setup, when the repository already has an initialized `orchestrator/` directory.

It is responsible for:

- loading persisted orchestration state
- resuming the current round or starting the next one
- delegating `select-task`, `plan`, `implement`, `review`, and `merge` stages to fresh subagents, preferring matching repo-local `.codex/agents/orchestrator-*.toml` role definitions and falling back to `orchestrator/roles/` when a role has not migrated yet
- updating only controller-owned state
- squash-merging approved rounds and advancing the roadmap

## Workflow Model

The runtime loop is strict about ownership:

- The orchestrator is a controller, not an implementer.
- Each round is linear and uses one branch plus one worktree.
- Review rejection sends the round back to `plan`, then `implement`, then `review` again.
- Resume behavior comes from files in the repository, not hidden session context.

The stage order is:

1. `select-task`
2. `plan`
3. `implement`
4. `review`
5. `merge`
6. `update-roadmap`
7. `done`

## Repo-Local Contract

The scaffolded repository gets a visible top-level `orchestrator/` directory plus repo-local role agents in `.codex/agents/`:

```text
.codex/
└── agents/
    ├── orchestrator-guider.toml
    ├── orchestrator-planner.toml
    ├── orchestrator-implementer.toml
    ├── orchestrator-reviewer.toml
    └── orchestrator-merger.toml

orchestrator/
├── state.json
├── rounds/
└── roadmaps/
    └── <roadmap_id>/
        └── <roadmap_revision>/
            ├── roadmap.md
            ├── verification.md
            └── retry-subloop.md
```

Key ideas behind that contract:

- `state.json` stays machine-oriented and tracks the active round, stage, branch, worktree, and resume errors.
- Human-facing reasoning stays in the active roadmap bundle `orchestrator/roadmaps/<roadmap_id>/<roadmap_revision>/roadmap.md`, repo-local role agents, and round artifacts.
- Each round folder stores delegated artifacts such as `selection.md`, `plan.md`, `implementation-notes.md`, `review.md`, and `merge.md`.
- The runtime skill prefers `.codex/agents/orchestrator-*.toml` per role and uses `orchestrator/roles/*.md` only when the matching repo-local agent file is missing.

## Repository Layout

This repository ships the skills under `skills/`:

```text
skills/
├── scaffold-orchestrator-loop/
│   ├── SKILL.md
│   ├── agents/openai.yaml
│   ├── assets/.codex/agents/
│   ├── assets/orchestrator/
│   └── references/
└── run-orchestrator-loop/
    ├── SKILL.md
    ├── agents/openai.yaml
    └── references/
```

- `SKILL.md` is the entrypoint instruction file.
- `references/` contains supporting rules for roadmap generation, state transitions, resume behavior, and merge boundaries.
- `assets/` contains the starter `orchestrator/` state plus repo-local orchestrator role agents used by the scaffold skill.

## Installation

Treat this repository as the source of truth and `~/.codex/skills` as the runtime install location.

To install directly from GitHub with the public `skills` CLI:

```bash
npx skills add https://github.com/soulomoon/orchestrator-skill --list
npx skills add https://github.com/soulomoon/orchestrator-skill --skill scaffold-orchestrator-loop
npx skills add https://github.com/soulomoon/orchestrator-skill --skill run-orchestrator-loop
```

For local development, prefer symlinks instead of copying the skill directories. A symlinked install updates immediately when the repo changes and avoids drift between the repo copy and the installed copy.

```bash
ln -s /path/to/orchestratorpattern/skills/scaffold-orchestrator-loop ~/.codex/skills/scaffold-orchestrator-loop
ln -s /path/to/orchestratorpattern/skills/run-orchestrator-loop ~/.codex/skills/run-orchestrator-loop
```

If either path already exists in `~/.codex/skills`, move or remove the existing entry before creating the symlink.

Copying the directories into `~/.codex/skills` also works, but it creates a second independent copy that can drift and must be resynced manually.

## Typical Usage

1. Install these skills in your Codex environment, either from GitHub with `npx skills add ...` or by symlinking from `skills/`.
2. In a target repository, invoke `scaffold-orchestrator-loop` with the high-level goal.
3. Review the generated `orchestrator/` contract and initial checkpoint commit.
4. Invoke `run-orchestrator-loop` to start or resume the delegated round loop.
5. Let the runtime skill continue until roadmap items are complete or a recorded controller error blocks progress.

## Development Notes

- Round branches are expected to use the `codex/` prefix.
- Round worktrees live under `.worktrees/` in the target repository.
- Reviewer approval is required before merge.
- Merge strategy is squash merge into the recorded base branch.

## Source Documents

The design and implementation plan used to build this repo are checked in under:

- `docs/superpowers/specs/2026-03-13-orchestrator-loop-design.md`
- `docs/superpowers/plans/2026-03-13-orchestrator-skill-set.md`
