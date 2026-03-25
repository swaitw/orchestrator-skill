# Revisioned Roadmap Skill Contract Implementation Plan

> [!INFO] Historical migration playbook
> This is the migration plan for moving from the legacy top-level contract to the current revisioned-roadmap model. It intentionally includes legacy-path gaps for comparison; these are historical references, not current operating instructions.

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Move `run-orchestrator-loop` and `scaffold-orchestrator-loop` to a revisioned-roadmap contract where live control files are resolved through `state.json` roadmap metadata instead of top-level `orchestrator/roadmap.md`, `orchestrator/verification.md`, and `orchestrator/retry-subloop.md`.

**Architecture:** The shared skill contract changes in one bounded slice: both skill docs, the runtime/reference docs they cite, and the scaffold assets/reference docs they generate. Runtime reads must become `state.json`-driven (`roadmap_id`, `roadmap_revision`, `roadmap_dir`), while scaffold output must create immutable `orchestrator/roadmaps/<roadmap_id>/rev-001/` bundles and require each round to record the active roadmap identity.

**Tech Stack:** Markdown skill docs, JSON scaffold asset, shell/grep verification

---

### Task 1: Freeze failing checks for the old top-level contract

**Files:**
- Modify: `/Users/ares/src/orchestratorpattern/docs/superpowers/plans/2026-03-25-revisioned-roadmap-skill-contract.md`
- Verify: `/Users/ares/src/orchestratorpattern/skills/public/run-orchestrator-loop/SKILL.md`
- Verify: `/Users/ares/src/orchestratorpattern/skills/public/run-orchestrator-loop/references/resume-rules.md`
- Verify: `/Users/ares/src/orchestratorpattern/skills/public/run-orchestrator-loop/references/worktree-merge-rules.md`
- Verify: `/Users/ares/src/orchestratorpattern/skills/public/scaffold-orchestrator-loop/SKILL.md`
- Verify: `/Users/ares/src/orchestratorpattern/skills/public/scaffold-orchestrator-loop/references/repo-contract.md`
- Verify: `/Users/ares/src/orchestratorpattern/skills/public/scaffold-orchestrator-loop/references/verification-contract.md`

- [ ] **Step 1: Run the failing-contract grep**

Run:
```bash
rg -n "orchestrator/roadmap\\.md|orchestrator/verification\\.md|orchestrator/retry-subloop\\.md|roadmap_id|roadmap_revision|roadmap_dir|orchestrator/roadmaps/" \
  /Users/ares/src/orchestratorpattern/skills/public/run-orchestrator-loop \
  /Users/ares/src/orchestratorpattern/skills/public/scaffold-orchestrator-loop
```

Expected: matches show the old top-level contract is still referenced and the new roadmap metadata fields are still absent from the shared-skill surface.

- [ ] **Step 2: Record the exact old-contract gaps in the plan**

Document that the current shared skills:
- still read top-level `orchestrator/roadmap.md`, `orchestrator/verification.md`, and `orchestrator/retry-subloop.md`;
- do not require `roadmap_id`, `roadmap_revision`, or `roadmap_dir`;
- do not scaffold `orchestrator/roadmaps/<roadmap_id>/rev-001/`.

### Task 2: Update scaffold-orchestrator-loop to generate revisioned roadmap bundles

**Files:**
- Modify: `/Users/ares/src/orchestratorpattern/skills/public/scaffold-orchestrator-loop/SKILL.md`
- Modify: `/Users/ares/src/orchestratorpattern/skills/public/scaffold-orchestrator-loop/references/repo-contract.md`
- Modify: `/Users/ares/src/orchestratorpattern/skills/public/scaffold-orchestrator-loop/references/roadmap-generation.md`
- Modify: `/Users/ares/src/orchestratorpattern/skills/public/scaffold-orchestrator-loop/references/verification-contract.md`
- Modify: `/Users/ares/src/orchestratorpattern/skills/public/scaffold-orchestrator-loop/assets/orchestrator/state.json`
- Delete: `/Users/ares/src/orchestratorpattern/skills/public/scaffold-orchestrator-loop/assets/orchestrator/roadmap.md`
- Delete: `/Users/ares/src/orchestratorpattern/skills/public/scaffold-orchestrator-loop/assets/orchestrator/verification.md`
- Add: `/Users/ares/src/orchestratorpattern/skills/public/scaffold-orchestrator-loop/assets/orchestrator/roadmaps/TEMPLATE_ROADMAP_ID/rev-001/roadmap.md`
- Add: `/Users/ares/src/orchestratorpattern/skills/public/scaffold-orchestrator-loop/assets/orchestrator/roadmaps/TEMPLATE_ROADMAP_ID/rev-001/retry-subloop.md`
- Add: `/Users/ares/src/orchestratorpattern/skills/public/scaffold-orchestrator-loop/assets/orchestrator/roadmaps/TEMPLATE_ROADMAP_ID/rev-001/verification.md`

- [ ] **Step 1: Write the failing asset-layout expectation**

Run:
```bash
find /Users/ares/src/orchestratorpattern/skills/public/scaffold-orchestrator-loop/assets/orchestrator -maxdepth 4 -type f | sort
```

Expected: current output lacks any `roadmaps/<roadmap_id>/rev-001/` directory and still exposes top-level `roadmap.md` and `verification.md`.

- [ ] **Step 2: Update the scaffold skill and references**

Make the scaffold contract require:
- `orchestrator/roadmaps/<roadmap_id>/<revision>/...` as the human-facing control-plane bundle;
- `state.json` fields `roadmap_id`, `roadmap_revision`, and `roadmap_dir`;
- immutable revisions once any round has used them;
- round packets recording roadmap identity in `selection.md` and `review-record.json`.

- [ ] **Step 3: Update the scaffold assets minimally**

Change `state.json` to include the roadmap metadata keys and point at the initial bundle. Replace the top-level roadmap/verification assets with a revisioned `rev-001` bundle containing:
- `roadmap.md`
- `retry-subloop.md`
- `verification.md`

- [ ] **Step 4: Run focused scaffold verification**

Run:
```bash
find /Users/ares/src/orchestratorpattern/skills/public/scaffold-orchestrator-loop/assets/orchestrator -maxdepth 5 -type f | sort
rg -n "roadmap_id|roadmap_revision|roadmap_dir|selection\\.md|review-record\\.json|orchestrator/roadmaps/.+/rev-001" \
  /Users/ares/src/orchestratorpattern/skills/public/scaffold-orchestrator-loop
```

Expected: scaffold assets expose the new revisioned bundle, and the skill/reference text requires the new roadmap metadata plus round-level roadmap identity recording.

### Task 3: Update run-orchestrator-loop to resolve the live contract through state.json roadmap metadata

**Files:**
- Modify: `/Users/ares/src/orchestratorpattern/skills/public/run-orchestrator-loop/SKILL.md`
- Modify: `/Users/ares/src/orchestratorpattern/skills/public/run-orchestrator-loop/references/state-machine.md`
- Modify: `/Users/ares/src/orchestratorpattern/skills/public/run-orchestrator-loop/references/resume-rules.md`
- Modify: `/Users/ares/src/orchestratorpattern/skills/public/run-orchestrator-loop/references/worktree-merge-rules.md`
- Modify: `/Users/ares/src/orchestratorpattern/skills/public/run-orchestrator-loop/references/delegation-boundaries.md`

- [ ] **Step 1: Write the failing runtime expectation**

Run:
```bash
rg -n "orchestrator/roadmap\\.md|orchestrator/verification\\.md|orchestrator/retry-subloop\\.md" \
  /Users/ares/src/orchestratorpattern/skills/public/run-orchestrator-loop
```

Expected: runtime docs still instruct the controller to read the top-level files directly.

- [ ] **Step 2: Rewrite runtime loading and resume rules**

Make the runtime contract require:
- reading `state.json` first;
- refusing to run when `roadmap_id`, `roadmap_revision`, or `roadmap_dir` is missing;
- resolving the live `roadmap.md`, `verification.md`, and `retry-subloop.md` through `roadmap_dir`;
- treating top-level orchestrator docs as non-authoritative for new repos;
- preserving the same roadmap identity through a round unless a merged `update-roadmap` stage lawfully activates a new revision.

- [ ] **Step 3: Keep merge/update-roadmap rules aligned**

Make merge bookkeeping and guider update rules say:
- merge finalization advances to `update-roadmap`;
- guider may activate a new revision only by updating `state.json` roadmap metadata;
- later roadmap checks use the active bundle from `roadmap_dir`, not a hard-coded top-level file path.

- [ ] **Step 4: Run focused runtime verification**

Run:
```bash
rg -n "roadmap_id|roadmap_revision|roadmap_dir|roadmap_dir/.+roadmap\\.md|roadmap_dir/.+verification\\.md|roadmap_dir/.+retry-subloop\\.md|top-level" \
  /Users/ares/src/orchestratorpattern/skills/public/run-orchestrator-loop
rg -n "orchestrator/roadmap\\.md|orchestrator/verification\\.md|orchestrator/retry-subloop\\.md" \
  /Users/ares/src/orchestratorpattern/skills/public/run-orchestrator-loop
```

Expected: the first grep shows the new metadata-driven runtime contract, and the second grep returns no stale direct-read instructions except any intentional historical context that is clearly labeled non-authoritative.

### Task 4: Final verification and summary

**Files:**
- Verify: `/Users/ares/src/orchestratorpattern/skills/public/run-orchestrator-loop`
- Verify: `/Users/ares/src/orchestratorpattern/skills/public/scaffold-orchestrator-loop`

- [ ] **Step 1: Run a whole-surface contract grep**

Run:
```bash
rg -n "roadmap_id|roadmap_revision|roadmap_dir|orchestrator/roadmaps/.+/rev-001|selection\\.md|review-record\\.json" \
  /Users/ares/src/orchestratorpattern/skills/public/run-orchestrator-loop \
  /Users/ares/src/orchestratorpattern/skills/public/scaffold-orchestrator-loop
```

Expected: both skills describe the same revisioned-roadmap contract.

- [ ] **Step 2: Run stale-contract grep**

Run:
```bash
rg -n "orchestrator/roadmap\\.md|orchestrator/verification\\.md|orchestrator/retry-subloop\\.md" \
  /Users/ares/src/orchestratorpattern/skills/public/run-orchestrator-loop \
  /Users/ares/src/orchestratorpattern/skills/public/scaffold-orchestrator-loop
```

Expected: no stale direct-live-contract references remain unless they are explicitly framed as deprecated or historical.

- [ ] **Step 3: Run diff hygiene**

Run:
```bash
git -C /Users/ares/src/orchestratorpattern diff --check
git -C /Users/ares/src/orchestratorpattern status --short
```

Expected: no whitespace/conflict issues; only the planned skill, reference, asset, and plan files are changed.
