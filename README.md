# Orchestrator Skill Set

This repository contains a pair of agent-neutral skills for running a strict repo-local orchestration workflow:

- `scaffold-orchestrator-loop` initializes an `orchestrator/` control plane in a target repository.
- `run-orchestrator-loop` resumes and coordinates the delegated round loop without doing the substantive work itself.

The design is intentionally explicit. State lives in the repository, delegated role definitions are inspectable, and each round uses a dedicated branch and worktree so progress can be resumed without relying on chat history.

## Included Skills

### `scaffold-orchestrator-loop`

Use this skill when a repository does not yet have an `orchestrator/` directory.

It is responsible for:

- surveying the repository and current goal
- generating the first roadmap
- scaffolding revisioned roadmap bundle files (for example `orchestrator/roadmaps/<roadmap_id>/<roadmap_revision>/roadmap.md`, `.../verification.md`, and `.../retry-subloop.md`, which stays present even when same-round retry is disabled), `orchestrator/state.json`, repo-local role definitions under `orchestrator/roles/`, and `orchestrator/worktrees/`
- preparing the repository for per-round worktrees
- creating the initial checkpoint commit

### `run-orchestrator-loop`

Use this skill after setup, when the repository already has an initialized `orchestrator/` directory.

It is responsible for:

- loading persisted orchestration state
- resuming the current round or starting the next one
- delegating `select-task`, `plan`, `implement`, `review`, and `merge` stages to fresh subagents that load their runtime instructions from `orchestrator/roles/`
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

The scaffolded repository gets a visible top-level `orchestrator/` directory with role definitions, rounds, roadmaps, and worktree staging kept in the repo:

```text
orchestrator/
├── state.json
├── roles/
│   ├── guider.md
│   ├── planner.md
│   ├── implementer.md
│   ├── reviewer.md
│   ├── merger.md
│   └── recovery-investigator.md
├── rounds/
├── roadmaps/
│   └── <roadmap_id>/
│       └── <roadmap_revision>/
│           ├── roadmap.md
│           ├── verification.md
│           └── retry-subloop.md
└── worktrees/
```

Key ideas behind that contract:

- `state.json` stays machine-oriented and tracks the active round, stage, branch, worktree, and resume errors.
- Human-facing reasoning stays in the active roadmap bundle `orchestrator/roadmaps/<roadmap_id>/<roadmap_revision>/roadmap.md`, repo-local role definitions, and round artifacts.
- Each round folder stores delegated artifacts such as `selection.md`, `plan.md`, `implementation-notes.md`, `review.md`, and `merge.md`.
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

Live validation on 2026-03-26 confirmed project-local installs for `claude-code`, `cursor`, `codex`, `gemini-cli`, `github-copilot`, and `opencode`. Global commands are intentionally omitted because the current `skills` CLI reported non-Claude global installs as `not linked`, so project-local installation is the documented supported path.
Run the `npx skills add ...` commands from the target repository root where these skills will be used.

### Claude Code

```bash
# project-local install
npx skills add https://github.com/soulomoon/orchestrator-skill --agent claude-code --skill scaffold-orchestrator-loop
npx skills add https://github.com/soulomoon/orchestrator-skill --agent claude-code --skill run-orchestrator-loop
# project-local verification
npx skills ls --agent claude-code
```

### Cursor

```bash
# project-local install
npx skills add https://github.com/soulomoon/orchestrator-skill --agent cursor --skill scaffold-orchestrator-loop
npx skills add https://github.com/soulomoon/orchestrator-skill --agent cursor --skill run-orchestrator-loop
# project-local verification
npx skills ls --agent cursor
```

### Codex

```bash
# project-local install
npx skills add https://github.com/soulomoon/orchestrator-skill --agent codex --skill scaffold-orchestrator-loop
npx skills add https://github.com/soulomoon/orchestrator-skill --agent codex --skill run-orchestrator-loop
# project-local verification
npx skills ls --agent codex
```

### Gemini CLI

```bash
# project-local install
npx skills add https://github.com/soulomoon/orchestrator-skill --agent gemini-cli --skill scaffold-orchestrator-loop
npx skills add https://github.com/soulomoon/orchestrator-skill --agent gemini-cli --skill run-orchestrator-loop
# project-local verification
npx skills ls --agent gemini-cli
```

### GitHub Copilot

```bash
# project-local install
npx skills add https://github.com/soulomoon/orchestrator-skill --agent github-copilot --skill scaffold-orchestrator-loop
npx skills add https://github.com/soulomoon/orchestrator-skill --agent github-copilot --skill run-orchestrator-loop
# project-local verification
npx skills ls --agent github-copilot
```

### OpenCode

```bash
# project-local install
npx skills add https://github.com/soulomoon/orchestrator-skill --agent opencode --skill scaffold-orchestrator-loop
npx skills add https://github.com/soulomoon/orchestrator-skill --agent opencode --skill run-orchestrator-loop
# project-local verification
npx skills ls --agent opencode
```

## Contributor Appendix

For local development, symlinks are still useful when you want a checked-out repo to act as the installed skill source. Keep this workflow as a contributor-only shortcut, not the primary install path. The block below is a host-specific Codex example using `~/.codex/skills`.

```bash
ln -s /path/to/orchestratorpattern/skills/scaffold-orchestrator-loop ~/.codex/skills/scaffold-orchestrator-loop
ln -s /path/to/orchestratorpattern/skills/run-orchestrator-loop ~/.codex/skills/run-orchestrator-loop
```

If either path already exists in `~/.codex/skills`, move or remove the existing entry before creating the symlink. Copying the directories into `~/.codex/skills` also works, but it creates a second independent copy that can drift and must be resynced manually.

## Typical Usage

1. From the target repository root, install these skills into that repository for your agent host with `npx skills add ... --agent <id>`.
2. In a target repository, invoke `scaffold-orchestrator-loop` with the high-level goal.
3. Review the generated `orchestrator/` contract and initial checkpoint commit.
4. Invoke `run-orchestrator-loop` to start or resume the delegated round loop.
5. Let the runtime skill continue until roadmap items are complete or a recorded controller error blocks progress.

## Development Notes

- Round branches are expected to use the `orchestrator/round-*` prefix.
- Round worktrees live under `orchestrator/worktrees/` in the target repository.
- Reviewer approval is required before merge.
- Merge strategy is squash merge into the recorded base branch.

## Source Documents

The design and implementation plan used to build this repo are checked in under:

- [docs/superpowers/specs/2026-03-26-platform-neutral-orchestrator-design.md](docs/superpowers/specs/2026-03-26-platform-neutral-orchestrator-design.md)
- [docs/superpowers/plans/2026-03-26-platform-neutral-orchestrator.md](docs/superpowers/plans/2026-03-26-platform-neutral-orchestrator.md)
