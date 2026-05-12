# Contributing

## Repository Structure

This repository ships two agent-neutral skills under `skills/`:

- `scaffold-orchestrator-loop` — initializes the `orchestrator/` control plane
- `run-orchestrator-loop` — resumes and coordinates the delegated round loop

Supporting files:
- `skills/*/references/` — supporting documentation loaded by the skills
- `skills/scaffold-orchestrator-loop/assets/` — templates copied into target repos

## Local Development

Symlink the skill directories into your agent's skill location for live
development. Replace `<skill-dir>` with your host agent's skill directory.

```bash
ln -s /path/to/orchestratorpattern/skills/scaffold-orchestrator-loop <skill-dir>/scaffold-orchestrator-loop
ln -s /path/to/orchestratorpattern/skills/run-orchestrator-loop <skill-dir>/run-orchestrator-loop
```

If either path already exists, move or remove the existing entry first.

## Validating Changes

After editing skill files, run these checks:

### Contract Integrity

```bash
# No host-specific tool references in active skill surfaces
! rg -n "request_user_input|\.codex/" \
  skills/run-orchestrator-loop \
  skills/scaffold-orchestrator-loop

# Canonical paths are present
rg -n "orchestrator/roles/|orchestrator/worktrees/|orchestrator/round-" \
  skills/run-orchestrator-loop/SKILL.md \
  skills/scaffold-orchestrator-loop/SKILL.md
```

### Skill Discovery

```bash
npx skills add . --list
# Expected: Found 2 skills
```

### Whitespace and Formatting

```bash
git diff --check
```
