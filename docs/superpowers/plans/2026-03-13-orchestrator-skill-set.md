# Orchestrator Skill Set Implementation Plan

> [!NOTE] Historical context
> This plan predates the revisioned roadmap contract and is kept for audit history. It references legacy top-level `orchestrator/roadmap.md` / `orchestrator/verification.md` paths and is not the current operating contract.

> **For agentic workers:** REQUIRED: Use superpowers:subagent-driven-development (if subagents available) or superpowers:executing-plans to implement this plan. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build two repo-local skills, `scaffold-orchestrator-loop` and `run-orchestrator-loop`, that implement the approved orchestrator contract and runtime loop.

**Architecture:** Keep the two skills self-contained under `skills/public`. The scaffold skill owns reusable reference docs plus template assets for the `orchestrator/` contract it generates inside target repos. The runtime skill owns the state-machine, delegation, resume, and merge guidance needed to coordinate the loop without doing substantive work itself.

**Tech Stack:** Markdown skill docs, repo-local reference and asset files, Python helper scripts from `/Users/ares/.codex/skills/.system/skill-creator/scripts`, Git

---

## File Structure

- Create: `skills/public/scaffold-orchestrator-loop/SKILL.md`
  Responsibility: entrypoint instructions for goal review, Git initialization, roadmap creation, repo-contract scaffolding, and initial checkpoint commit.
- Create: `skills/public/scaffold-orchestrator-loop/agents/openai.yaml`
  Responsibility: UI metadata generated from the skill content.
- Create: `skills/public/scaffold-orchestrator-loop/references/repo-contract.md`
  Responsibility: exact `orchestrator/` directory contract, state schema, round artifact rules, and required files.
- Create: `skills/public/scaffold-orchestrator-loop/references/roadmap-generation.md`
  Responsibility: how to derive an actionable roadmap from the goal plus repo survey.
- Create: `skills/public/scaffold-orchestrator-loop/references/verification-contract.md`
  Responsibility: how to define canonical reviewer checks and when to add task-specific verification.
- Create: `skills/public/scaffold-orchestrator-loop/assets/orchestrator/roadmap.md`
  Responsibility: starter roadmap template copied into target repos.
- Create: `skills/public/scaffold-orchestrator-loop/assets/orchestrator/state.json`
  Responsibility: starter machine-state template copied into target repos.
- Create: `skills/public/scaffold-orchestrator-loop/assets/orchestrator/verification.md`
  Responsibility: starter verification contract copied into target repos.
- Create: `skills/public/scaffold-orchestrator-loop/assets/orchestrator/roles/guider.md`
- Create: `skills/public/scaffold-orchestrator-loop/assets/orchestrator/roles/planner.md`
- Create: `skills/public/scaffold-orchestrator-loop/assets/orchestrator/roles/implementer.md`
- Create: `skills/public/scaffold-orchestrator-loop/assets/orchestrator/roles/reviewer.md`
- Create: `skills/public/scaffold-orchestrator-loop/assets/orchestrator/roles/merger.md`
  Responsibility: inspectable starter role prompts copied into target repos.
- Create: `skills/public/run-orchestrator-loop/SKILL.md`
  Responsibility: coordinator-only runtime instructions for round startup, resume, delegation, review loop, and merge completion.
- Create: `skills/public/run-orchestrator-loop/agents/openai.yaml`
  Responsibility: UI metadata generated from the runtime skill content.
- Create: `skills/public/run-orchestrator-loop/references/state-machine.md`
  Responsibility: allowed stage transitions and controller-only responsibilities.
- Create: `skills/public/run-orchestrator-loop/references/resume-rules.md`
  Responsibility: how to resume the exact active stage after interruption or rejection.
- Create: `skills/public/run-orchestrator-loop/references/delegation-boundaries.md`
  Responsibility: what the orchestrator may edit directly versus what must come from delegated agents.
- Create: `skills/public/run-orchestrator-loop/references/worktree-merge-rules.md`
  Responsibility: branch naming, worktree reuse, squash-merge, and round finalization rules.

## Chunk 1: Bootstrap Skill Skeletons

### Task 1: Initialize the scaffold skill directory

**Files:**
- Create: `skills/public/scaffold-orchestrator-loop/SKILL.md`
- Create: `skills/public/scaffold-orchestrator-loop/agents/openai.yaml`
- Create: `skills/public/scaffold-orchestrator-loop/references/`
- Create: `skills/public/scaffold-orchestrator-loop/assets/`

- [ ] **Step 1: Run the scaffold-skill initializer**

Run:

```bash
python3 /Users/ares/.codex/skills/.system/skill-creator/scripts/init_skill.py \
  scaffold-orchestrator-loop \
  --path skills/public \
  --resources references,assets \
  --interface display_name="Scaffold Orchestrator Loop" \
  --interface short_description="Create roadmap, control files, and role prompts for an orchestrator repo loop." \
  --interface default_prompt="Scaffold an orchestrator loop for this repository from the current goal."
```

Expected: `skills/public/scaffold-orchestrator-loop/` exists with `SKILL.md`, `agents/openai.yaml`, `references/`, and `assets/`.

- [ ] **Step 2: Inspect the generated scaffold files**

Run:

```bash
find skills/public/scaffold-orchestrator-loop -maxdepth 3 -type f | sort
```

Expected: only the generated skill files appear, with no missing `agents/openai.yaml`.

### Task 2: Initialize the runtime skill directory

**Files:**
- Create: `skills/public/run-orchestrator-loop/SKILL.md`
- Create: `skills/public/run-orchestrator-loop/agents/openai.yaml`
- Create: `skills/public/run-orchestrator-loop/references/`

- [ ] **Step 1: Run the runtime-skill initializer**

Run:

```bash
python3 /Users/ares/.codex/skills/.system/skill-creator/scripts/init_skill.py \
  run-orchestrator-loop \
  --path skills/public \
  --resources references \
  --interface display_name="Run Orchestrator Loop" \
  --interface short_description="Resume and coordinate a delegated roadmap loop without doing the work directly." \
  --interface default_prompt="Run or resume the orchestrator loop for this repository."
```

Expected: `skills/public/run-orchestrator-loop/` exists with `SKILL.md`, `agents/openai.yaml`, and `references/`.

- [ ] **Step 2: Inspect the generated runtime files**

Run:

```bash
find skills/public/run-orchestrator-loop -maxdepth 3 -type f | sort
```

Expected: the generated runtime skill files appear and no unexpected placeholder assets exist.

### Task 3: Check the bootstrap state before writing custom content

**Files:**
- Modify: `skills/public/scaffold-orchestrator-loop/SKILL.md`
- Modify: `skills/public/run-orchestrator-loop/SKILL.md`

- [ ] **Step 1: Open both generated SKILL templates and note the placeholder regions**

Run:

```bash
sed -n '1,220p' skills/public/scaffold-orchestrator-loop/SKILL.md
sed -n '1,220p' skills/public/run-orchestrator-loop/SKILL.md
```

Expected: both files still contain the template sections that need replacing.

- [ ] **Step 2: Commit the generated skeletons**

Run:

```bash
git add skills/public
git commit -m "chore: scaffold orchestrator skill directories"
```

Expected: Git records the empty skill skeletons before custom content is added.

## Chunk 2: Build the Scaffold Skill

### Task 4: Write scaffold reference docs

**Files:**
- Create: `skills/public/scaffold-orchestrator-loop/references/repo-contract.md`
- Create: `skills/public/scaffold-orchestrator-loop/references/roadmap-generation.md`
- Create: `skills/public/scaffold-orchestrator-loop/references/verification-contract.md`

- [ ] **Step 1: Write `repo-contract.md` with the required `orchestrator/` layout**

Include:

```markdown
# Repo Contract

## Required Files
- orchestrator/roadmap.md
- orchestrator/state.json
- orchestrator/verification.md
- orchestrator/roles/*.md
- orchestrator/rounds/

## state.json keys
| key | type | meaning |
| --- | --- | --- |
| active_round_id | string or null | current round identifier |
| stage | string | one of select-task, plan, implement, review, merge, update-roadmap, done |
| branch | string or null | round branch name |
| worktree_path | string or null | active round worktree |
```

Expected: the reference names every required file from the approved spec and gives the state schema the scaffold skill must emit.

- [ ] **Step 2: Write `roadmap-generation.md` with roadmap derivation rules**

Include explicit rules for:
- surveying the repo before drafting tasks,
- decomposing the goal into sequential roadmap items,
- recording status, dependencies, and completion notes,
- avoiding speculative future rounds beyond the next actionable roadmap items.

Expected: the reference tells the scaffold skill how to produce the first roadmap without inventing implementation details prematurely.

- [ ] **Step 3: Write `verification-contract.md` with reviewer expectations**

Include:
- required baseline verification sections,
- how to document repo-specific verification commands,
- when to add task-specific checks,
- how runtime reviewers should record pass or fail outcomes.

Expected: the reference gives the scaffold skill enough structure to generate a reusable `orchestrator/verification.md`.

- [ ] **Step 4: Review the three reference files together**

Run:

```bash
sed -n '1,220p' skills/public/scaffold-orchestrator-loop/references/repo-contract.md
sed -n '1,220p' skills/public/scaffold-orchestrator-loop/references/roadmap-generation.md
sed -n '1,220p' skills/public/scaffold-orchestrator-loop/references/verification-contract.md
```

Expected: the references read as a coherent scaffold workflow with no TODOs or duplicated rules.

### Task 5: Create scaffold asset templates

**Files:**
- Create: `skills/public/scaffold-orchestrator-loop/assets/orchestrator/roadmap.md`
- Create: `skills/public/scaffold-orchestrator-loop/assets/orchestrator/state.json`
- Create: `skills/public/scaffold-orchestrator-loop/assets/orchestrator/verification.md`
- Create: `skills/public/scaffold-orchestrator-loop/assets/orchestrator/roles/guider.md`
- Create: `skills/public/scaffold-orchestrator-loop/assets/orchestrator/roles/planner.md`
- Create: `skills/public/scaffold-orchestrator-loop/assets/orchestrator/roles/implementer.md`
- Create: `skills/public/scaffold-orchestrator-loop/assets/orchestrator/roles/reviewer.md`
- Create: `skills/public/scaffold-orchestrator-loop/assets/orchestrator/roles/merger.md`

- [ ] **Step 1: Write the roadmap template**

Include:

```markdown
# Roadmap

## Status Legend
- pending
- in-progress
- done

## Items
1. [pending] Example task
   Depends on:
   Completion notes:
```

Expected: the template shows the exact status format the guider will maintain later.

- [ ] **Step 2: Write the initial `state.json` template**

Include:

```json
{
  "active_round_id": null,
  "stage": "done",
  "current_task": null,
  "branch": null,
  "worktree_path": null,
  "round_artifacts": {},
  "last_completed_round": null
}
```

Expected: the JSON is valid and matches the state schema from the spec.

- [ ] **Step 3: Write the verification template**

Include sections for:
- baseline commands,
- task-specific checks,
- approval criteria,
- reviewer recording format.

Expected: the template is specific enough that scaffolded repos can fill in real commands immediately.

- [ ] **Step 4: Write the five role prompt templates**

Each role file must encode the approved boundaries:
- `guider.md`: owns `select-task` and roadmap updates.
- `planner.md`: writes the round plan and revises it after rejection.
- `implementer.md`: applies the plan in the round worktree.
- `reviewer.md`: runs canonical plus change-specific verification and approves or rejects.
- `merger.md`: prepares merge notes for accepted rounds.

Expected: each role file has one job, no overlap, and no instruction that lets the orchestrator do substantive work.

- [ ] **Step 5: Inspect the asset tree**

Run:

```bash
find skills/public/scaffold-orchestrator-loop/assets -maxdepth 4 -type f | sort
```

Expected: all contract templates and role templates are present in the asset tree.

- [ ] **Step 6: Commit the scaffold references and assets**

Run:

```bash
git add skills/public/scaffold-orchestrator-loop
git commit -m "feat: add scaffold orchestrator skill resources"
```

Expected: scaffold references and templates are committed separately from the runtime skill.

### Task 6: Replace the scaffold SKILL.md template with final instructions

**Files:**
- Modify: `skills/public/scaffold-orchestrator-loop/SKILL.md`
- Modify: `skills/public/scaffold-orchestrator-loop/agents/openai.yaml`

- [ ] **Step 1: Write the scaffold skill frontmatter**

Use:

```yaml
---
name: scaffold-orchestrator-loop
description: Create a repo-local orchestrator loop from a goal by reviewing the repository, initializing Git if needed, building the first roadmap, scaffolding `orchestrator/` control files, and writing inspectable role prompts. Use when starting a new delegated orchestrator workflow or when a repository needs the loop state initialized before runtime orchestration.
---
```

Expected: the description names both the capability and the trigger conditions.

- [ ] **Step 2: Write the scaffold workflow body**

The body must instruct the skill to:
- review the goal and repository first,
- read the scaffold references when generating roadmap and verification content,
- copy the asset templates into the target repo's `orchestrator/`,
- initialize Git if missing,
- make the initial checkpoint commit after scaffolding,
- stop after setup rather than running implementation rounds.

Expected: the body stays concise and pushes durable detail into the reference files.

- [ ] **Step 3: Regenerate `agents/openai.yaml`**

Run:

```bash
python3 /Users/ares/.codex/skills/.system/skill-creator/scripts/generate_openai_yaml.py \
  skills/public/scaffold-orchestrator-loop \
  --interface display_name="Scaffold Orchestrator Loop" \
  --interface short_description="Create roadmap, control files, and role prompts for an orchestrator repo loop." \
  --interface default_prompt="Scaffold an orchestrator loop for this repository from the current goal."
```

Expected: `agents/openai.yaml` reflects the final frontmatter and interface strings.

- [ ] **Step 4: Validate the scaffold skill**

Run:

```bash
python3 /Users/ares/.codex/skills/.system/skill-creator/scripts/quick_validate.py \
  skills/public/scaffold-orchestrator-loop
```

Expected: validation succeeds with no YAML or naming errors.

- [ ] **Step 5: Commit the finished scaffold skill**

Run:

```bash
git add skills/public/scaffold-orchestrator-loop
git commit -m "feat: finalize scaffold orchestrator skill"
```

Expected: the scaffold skill is complete and validated before the runtime skill is written.

## Chunk 3: Build the Runtime Skill

### Task 7: Write runtime reference docs

**Files:**
- Create: `skills/public/run-orchestrator-loop/references/state-machine.md`
- Create: `skills/public/run-orchestrator-loop/references/resume-rules.md`
- Create: `skills/public/run-orchestrator-loop/references/delegation-boundaries.md`
- Create: `skills/public/run-orchestrator-loop/references/worktree-merge-rules.md`

- [ ] **Step 1: Write `state-machine.md`**

Include:

```markdown
# State Machine

1. select-task
2. plan
3. implement
4. review
5. merge
6. update-roadmap
7. done
```

Also include the legal transitions and the rule that `select-task` is delegated to the guider, not performed by the orchestrator.

Expected: the reference captures the exact round lifecycle from the approved spec.

- [ ] **Step 2: Write `resume-rules.md`**

Cover:
- automatic resume from `state.json`,
- reuse of the same round id, branch, and worktree after rejection,
- restarting the exact incomplete stage,
- what to do when a recorded worktree is missing or state is corrupt.

Expected: the runtime skill has a crisp resume contract instead of improvising.

- [ ] **Step 3: Write `delegation-boundaries.md`**

Spell out:
- orchestrator may update only machine-control files,
- planner, implementer, reviewer, and merger author round artifacts,
- runtime must never do the real work itself,
- subagents must be fresh per stage and per round,
- runtime must not interrupt a live subagent or set a timeout on it.

Expected: the user's delegation rules are encoded in one place and referenced from `SKILL.md`.

- [ ] **Step 4: Write `worktree-merge-rules.md`**

Cover:
- one branch and one worktree per round,
- branch naming convention,
- squash merge after reviewer approval,
- roadmap update and round closure after merge.

Expected: the runtime skill has deterministic round-finalization rules.

- [ ] **Step 5: Review the runtime references together**

Run:

```bash
sed -n '1,220p' skills/public/run-orchestrator-loop/references/state-machine.md
sed -n '1,220p' skills/public/run-orchestrator-loop/references/resume-rules.md
sed -n '1,220p' skills/public/run-orchestrator-loop/references/delegation-boundaries.md
sed -n '1,220p' skills/public/run-orchestrator-loop/references/worktree-merge-rules.md
```

Expected: the runtime references agree on stage order, role ownership, and controller boundaries.

### Task 8: Replace the runtime SKILL.md template with final instructions

**Files:**
- Modify: `skills/public/run-orchestrator-loop/SKILL.md`
- Modify: `skills/public/run-orchestrator-loop/agents/openai.yaml`

- [ ] **Step 1: Write the runtime skill frontmatter**

Use:

```yaml
---
name: run-orchestrator-loop
description: Run or resume a repo-local orchestrator loop by reading `orchestrator/` state, delegating task selection to the guider, coordinating planner, implementer, reviewer, and merger agents in a strict linear round, resuming interrupted stages automatically, and squash-merging accepted rounds. Use when a repository already has an initialized orchestrator contract and needs the delegated loop executed or resumed.
---
```

Expected: the frontmatter triggers only once setup has already happened.

- [ ] **Step 2: Write the runtime workflow body**

The body must instruct the skill to:
- load `orchestrator/state.json` first,
- delegate `select-task` to the guider,
- create or reuse the round branch and worktree,
- spawn fresh stage-specific subagents,
- loop planner -> implementer -> reviewer until approval,
- keep the same round active across rejections,
- squash-merge only after reviewer approval,
- update only machine-control state directly.

Expected: the body makes the orchestrator act as a strict state machine over repo-local state.

- [ ] **Step 3: Add the non-interruption rule**

Add explicit instructions that the runtime skill must:
- never cancel a live subagent,
- never set a timeout for a live subagent,
- wait for the subagent to finish before continuing,
- remain coordinator-only while that agent is active.

Expected: the runtime skill directly encodes the user's subagent-handling rule instead of leaving it implicit.

- [ ] **Step 4: Regenerate `agents/openai.yaml`**

Run:

```bash
python3 /Users/ares/.codex/skills/.system/skill-creator/scripts/generate_openai_yaml.py \
  skills/public/run-orchestrator-loop \
  --interface display_name="Run Orchestrator Loop" \
  --interface short_description="Resume and coordinate a delegated roadmap loop without doing the work directly." \
  --interface default_prompt="Run or resume the orchestrator loop for this repository."
```

Expected: `agents/openai.yaml` matches the runtime skill behavior and triggers.

- [ ] **Step 5: Validate the runtime skill**

Run:

```bash
python3 /Users/ares/.codex/skills/.system/skill-creator/scripts/quick_validate.py \
  skills/public/run-orchestrator-loop
```

Expected: validation succeeds with no YAML or naming errors.

- [ ] **Step 6: Commit the finished runtime skill**

Run:

```bash
git add skills/public/run-orchestrator-loop
git commit -m "feat: finalize runtime orchestrator skill"
```

Expected: the runtime skill lands as a separate commit from the scaffold skill.

### Task 9: Validate both skills together

**Files:**
- Modify: `skills/public/scaffold-orchestrator-loop/agents/openai.yaml`
- Modify: `skills/public/run-orchestrator-loop/agents/openai.yaml`

- [ ] **Step 1: Run validation for both skills**

Run:

```bash
python3 /Users/ares/.codex/skills/.system/skill-creator/scripts/quick_validate.py \
  skills/public/scaffold-orchestrator-loop
python3 /Users/ares/.codex/skills/.system/skill-creator/scripts/quick_validate.py \
  skills/public/run-orchestrator-loop
```

Expected: both validations pass back-to-back.

- [ ] **Step 2: Inspect the generated metadata files**

Run:

```bash
sed -n '1,220p' skills/public/scaffold-orchestrator-loop/agents/openai.yaml
sed -n '1,220p' skills/public/run-orchestrator-loop/agents/openai.yaml
```

Expected: display name, short description, and default prompt reflect the final skill copy and not the initializer placeholders.

## Chunk 4: Smoke Test and Finish

### Task 10: Smoke-test the scaffold skill in a disposable fixture repo

**Files:**
- Test: `/tmp/orchestrator-skill-smoke/scaffold-target/`

- [ ] **Step 1: Create a disposable fixture repo**

Run:

```bash
rm -rf /tmp/orchestrator-skill-smoke
mkdir -p /tmp/orchestrator-skill-smoke/scaffold-target
```

Expected: the fixture path exists and is empty.

- [ ] **Step 2: Invoke the scaffold skill against the fixture**

If subagents are available, use a fresh worker with a prompt equivalent to:

```text
Use $scaffold-orchestrator-loop at /Users/ares/src/orchestratorpattern/skills/public/scaffold-orchestrator-loop to scaffold an orchestrator loop in /tmp/orchestrator-skill-smoke/scaffold-target for the goal: create a README-only repository improvement workflow.
```

If subagents are not available, manually follow the skill instructions in the fixture repo.

Expected: the fixture gets `orchestrator/roadmap.md`, `state.json`, `verification.md`, `roles/`, and an initial Git commit.

- [ ] **Step 3: Inspect the scaffolded contract**

Run:

```bash
find /tmp/orchestrator-skill-smoke/scaffold-target/orchestrator -maxdepth 3 -type f | sort
git -C /tmp/orchestrator-skill-smoke/scaffold-target log --oneline -1
```

Expected: all required contract files exist and the repo has a checkpoint commit.

- [ ] **Step 4: Fix any scaffold-skill issues discovered in the smoke test**

Expected: any missing file, bad template, or unclear instruction is corrected before moving on.

### Task 11: Smoke-test the runtime skill on the scaffolded fixture

**Files:**
- Test: `/tmp/orchestrator-skill-smoke/scaffold-target/`

- [ ] **Step 1: Seed the fixture with a minimal actionable goal**

Create a single roadmap item such as adding `README.md` so the runtime loop has a trivial round to complete.

Expected: the fixture is small enough that a single round can finish quickly and exercise the runtime loop.

- [ ] **Step 2: Invoke the runtime skill in a fresh worker**

Use a prompt equivalent to:

```text
Use $run-orchestrator-loop at /Users/ares/src/orchestratorpattern/skills/public/run-orchestrator-loop to run or resume the orchestrator loop in /tmp/orchestrator-skill-smoke/scaffold-target until the active roadmap item is complete.
```

Expected: the runtime creates one round branch and worktree, delegates the stages, merges the accepted round, and updates `orchestrator/state.json`.

- [ ] **Step 3: Verify round completion**

Run:

```bash
git -C /tmp/orchestrator-skill-smoke/scaffold-target log --oneline -3
sed -n '1,220p' /tmp/orchestrator-skill-smoke/scaffold-target/orchestrator/state.json
sed -n '1,220p' /tmp/orchestrator-skill-smoke/scaffold-target/orchestrator/roadmap.md
```

Expected: the latest history includes one accepted round, the roadmap item is marked done, and state shows no active round.

- [ ] **Step 4: Patch any runtime-skill issues discovered in the smoke test**

Expected: resume errors, delegation leaks, or merge-state bugs are corrected before final validation.

### Task 12: Revalidate, review, and ship the skill set

**Files:**
- Modify: `skills/public/scaffold-orchestrator-loop/SKILL.md`
- Modify: `skills/public/run-orchestrator-loop/SKILL.md`

- [ ] **Step 1: Re-run both validators after smoke-test fixes**

Run:

```bash
python3 /Users/ares/.codex/skills/.system/skill-creator/scripts/quick_validate.py \
  skills/public/scaffold-orchestrator-loop
python3 /Users/ares/.codex/skills/.system/skill-creator/scripts/quick_validate.py \
  skills/public/run-orchestrator-loop
```

Expected: both skills still validate after smoke-test adjustments.

- [ ] **Step 2: Inspect the final diff**

Run:

```bash
git status --short
git diff --stat
```

Expected: only the intended skill folders and related plan/spec documents are changed.

- [ ] **Step 3: Commit the final tested skill set**

Run:

```bash
git add skills/public docs/superpowers/plans/2026-03-13-orchestrator-skill-set.md
git commit -m "feat: add orchestrator skill set"
```

Expected: the repo records a final commit containing the plan and the completed skills.
