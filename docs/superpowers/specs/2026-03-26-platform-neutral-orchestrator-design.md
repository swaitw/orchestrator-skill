# Platform-Neutral Orchestrator Skill Design

## Goal

Convert the orchestrator skill set from a Codex-shaped workflow into a
platform-neutral workflow that installs cleanly from `skills.sh` and runs
against a single canonical repo-local contract regardless of the host agent.

This design intentionally removes `.codex/agents/` rather than keeping it as
an adapter or compatibility layer.

## Current Problems

- The repo describes itself as a Codex-specific skill set.
- The scaffolded repo contract depends on `.codex/agents/`.
- The runtime skill prefers `.codex/agents/orchestrator-<role>.toml` and only
  falls back to `orchestrator/roles/<role>.md`.
- Installation guidance is written around `~/.codex/skills`.
- The round branch prefix `codex/` leaks a host-agent identity into a repo
  contract that should be portable.

These constraints make the skills installable through `skills.sh`, but the
actual runtime contract remains tied to one host environment.

## Decision

The only canonical repo-local role source will be:

- `orchestrator/roles/<role>.md`

The repo will no longer mention, scaffold, prefer, or fall back to
`.codex/agents/` in any active skill, asset, contract, or user-facing
documentation surface.

Legacy references may remain only in explicitly historical documents that are
describing the retired contract rather than defining the live one.

## Target Contract

After the rewrite, a scaffolded repository should look like:

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
├── worktrees/
└── roadmaps/
    └── <roadmap_id>/
        └── <roadmap_revision>/
            ├── roadmap.md
            ├── verification.md
            └── retry-subloop.md
```

The runtime skill must load role instructions only from `orchestrator/roles/`.
If a required role file is missing, the controller should stop with an exact
controller error instead of inventing role behavior.

Per-round worktrees must live under `orchestrator/worktrees/` rather than
`.worktrees/`.

`recovery-investigator` is part of the canonical repo-local contract and must
live at `orchestrator/roles/recovery-investigator.md`. It is not a shared
skill-owned external recovery helper in the target design.

## Skill Changes

### `scaffold-orchestrator-loop`

The scaffold skill will:

- copy `assets/orchestrator` into the target repo root as `orchestrator/`
- scaffold role definitions into `orchestrator/roles/`
- stop mentioning `.codex/agents/` in descriptions, workflow steps, and
  resource references
- stop making git-ignore exceptions for `.codex/`
- ensure `orchestrator/worktrees/` is ignored by a tracked ignore rule unless
  the repository already ignores it through a broader rule

Asset changes:

- remove `assets/.codex/agents/`
- add role templates under `assets/orchestrator/roles/`, including
  `recovery-investigator.md`

### `run-orchestrator-loop`

The runtime skill will:

- load role definitions only from `orchestrator/roles/`
- remove `.codex/agents/` lookup and fallback logic
- keep the existing controller-only ownership model
- preserve the existing recovery-investigator requirement through the new
  repo-local `orchestrator/roles/recovery-investigator.md` contract

Reference changes:

- rewrite any references that prefer `.codex/agents/`
- rewrite any migration or compatibility text that treats
  `orchestrator/roles/` as secondary
- rewrite recovery rules so `recovery-investigator` is resolved from
  `orchestrator/roles/` like every other delegated role

## Naming and Branching

The branch prefix becomes platform-neutral and mandatory:

- `orchestrator/round-<nn>-<slug>`

This keeps branch names clearly associated with the orchestrator workflow while
removing Codex-specific branding.

Existing `codex/round-*` branches and already-scaffolded repositories using the
old contract are intentionally unsupported after this rewrite. The skill set
will not carry a compatibility or migration layer for the old branch prefix,
`.codex/agents/`, or `.worktrees/`.

## README Changes

The README should be rewritten to describe these as agent skills rather than
Codex skills.

Required documentation changes:

- remove Codex-specific identity language
- describe `orchestrator/roles/` as the canonical runtime contract
- describe `orchestrator/worktrees/` as the canonical worktree location
- remove `~/.codex/skills` as the primary installation framing
- keep the repository layout section aligned with the actual skill folders
- add installation examples for the mainstream host agents the project wants to
  support

## Install Surface

The README should include short install subsections for:

- Claude Code
- Cursor
- Codex
- Gemini CLI
- GitHub Copilot
- OpenCode

Each subsection should include:

- a project-local install command
- a global install command if the CLI supports it consistently
- a minimal verification command or usage check

The commands should use the official `skills` CLI agent flags documented by
the `vercel-labs/skills` project, not repo-specific symlink instructions as the
primary onboarding path.

These six agents are documented install targets, not a promise of host-specific
runtime behavior beyond the shared platform-neutral contract. Verification
should confirm that the install commands and skill discovery paths remain
correct; it does not need to certify every downstream host-agent UX detail.

Symlink-based local development instructions may remain as a secondary section
for contributors.

## Error Handling

The rewrite must preserve the current controller guarantees:

- no delegated work in the controller voice
- no silent fallback to invented role behavior
- no terminal stop for non-terminal delegated-stage failure until
  the repo-local `recovery-investigator` is attempted or deterministically
  unavailable
- no guessing when state or role definitions are missing

## Testing and Verification

Because this repository is documentation- and skill-contract-heavy rather than
code-heavy, verification should focus on observable contract integrity:

- `git diff --check`
- `rg` checks showing `.codex/agents/` has been removed from all active skill,
  asset, contract, and README surfaces
- `rg` checks showing active branch-prefix references use
  `orchestrator/round-` rather than `codex/round-`
- `npx skills add . --list` confirming both skills are still discoverable
- local `skills` CLI checks for the documented agent flags when available
- validation that the six documented install targets still match the current
  `skills` CLI agent identifiers before finalizing README commands
- one remote `npx skills add <repo-url> --list` check when network conditions
  allow it

## Implementation Plan

The implementation should proceed in this order:

1. Rewrite scaffold assets so role templates live under
   `assets/orchestrator/roles/`.
2. Rewrite `scaffold-orchestrator-loop` to scaffold only the new role path.
3. Rewrite `run-orchestrator-loop` and its references to use only
   `orchestrator/roles/`.
4. Rewrite README language and installation sections.
5. Verify skill discovery and contract integrity.

## Risks

- Missing one `.codex/agents/` reference would leave the repo in an ambiguous
  partially migrated state.
- Agent install examples can drift if the upstream `skills` CLI changes agent
  identifiers.
- README examples that claim support for a host agent without actual install
  verification will weaken trust.

## Non-Goals

- Building per-agent runtime adapters
- Keeping `.codex/agents/` as a compatibility path
- Adding host-specific orchestration behavior
- Creating a marketplace- or plugin-specific distribution system beyond the
  standard `skills` CLI install path
