# Alignment Brainstorm

Use this before drafting a fresh roadmap family. The purpose is to turn a
rough goal into an approved, repo-specific orchestration strategy that can be
persisted in the control plane.

This phase adapts the Superpowers brainstorming pattern for orchestrator
scaffolding: understand the project context, ask focused questions, compare
approaches, get approval, then write the approved direction into durable
orchestrator files.

## Required Inputs

- User's initial goal
- Repository survey notes
- Existing docs, tests, and commands
- Existing terminal orchestrator state when using `next-family`
- Prior active roadmap bundle and `project-contract.md` when present

## Process

1. Explore the repo enough to understand current architecture, tests, risks,
   and existing workflow conventions.
2. Check whether the user goal already answers the core alignment questions.
3. Ask only the missing high-signal questions through the decision loop below.
4. If the goal is too large for one roadmap family, decompose it first and ask
   which family should be scaffolded now.
5. Propose 2-3 roadmap strategies with tradeoffs and a recommendation.
6. Present the recommended strategy for approval before writing scaffold files.
7. If the user changes direction, revise the strategy and ask for approval
   again.
8. After approval, draft the roadmap family from the approved alignment rather
   than from the initial prompt alone.

Do not create or update `orchestrator/` files until the user has approved the
alignment strategy, unless the user explicitly provided a complete approved
strategy in the request.

## Core Alignment Questions

Cover these by reading the repo, reading the user's goal, or asking concise
follow-up questions:

- What outcome should this roadmap family deliver?
- What is out of scope even if it is adjacent?
- What would make the work meaningfully complete for a human reviewer?
- Which architecture or compatibility constraints must be preserved?
- Which tests, builds, demos, or manual checks define acceptable progress?
- What risks should affect sequencing?
- Which work fronts are safe to run concurrently, and which must stay serial?
- What tradeoffs does the user prefer when speed, scope, and design quality
  conflict?

Ask one question at a time through the decision loop when conversational
alignment is needed. Use multiple choice by default; open-ended questions are
only for missing context that cannot be honestly bounded into useful options.

## Decision Loop

For each unresolved alignment decision:

1. Ask exactly one question.
2. Provide 2-3 labeled options whenever the decision can be bounded.
3. Put the recommended option first and say why it fits the repo and goal.
4. State the tradeoff for each option.
5. Wait for the user's answer before asking the next question.
6. Record the selected answer in the alignment decision ledger.

If an open-ended question is unavoidable, ask exactly one open-ended question
and do not include any other decision in the same message.

## Structured Choice UI

When the host exposes a structured choice tool, use it for every bounded
alignment decision instead of plain text options. Either call the host tool or
use the explicit text fallback below. Do not simulate a clickable picker with
Markdown.

If the host does not expose a structured choice tool in the current mode, say
so, then ask the same one-question decision with text `A` / `B` / `C` options.
Do not tell the user they can click unless a real host picker has been launched.

Structured choice prompts must:

- ask exactly one decision question;
- provide 2-3 mutually exclusive choices;
- put the recommended choice first and mark it as recommended in the label;
- describe the impact or tradeoff of each choice in one sentence;
- allow the host's free-form fallback only for genuinely unlisted answers; and
- wait for the user's selection before continuing.

Use text `A` / `B` / `C` options only when structured choice UI is unavailable
or the decision cannot be honestly bounded. Do not skip the choice step by
presenting a strategy summary as if it were already approved.

Use this shape for bounded decisions:

```md
### Alignment Decision: <short topic>

Recommended: A

A. <option title>
Impact: <what this chooses>
Tradeoff: <what this costs or defers>

B. <option title>
Impact: <what this chooses>
Tradeoff: <what this costs or defers>

C. <option title>
Impact: <what this chooses>
Tradeoff: <what this costs or defers>

Which direction should this roadmap use?
```

Use an open-ended question only when the missing context cannot be honestly
bounded into useful options. Do not batch several decisions into one question.

## Strategy Options

Before approval, present 2-3 possible roadmap strategies. Each option should
name:

- the organizing thesis
- the first milestone shape
- sequencing and dependency consequences
- concurrency posture
- main risk

Lead with the recommended option and explain why it fits the repo and user
goal. Keep options short enough for the user to compare directly.

## Alignment Decision Ledger

Before drafting the roadmap, summarize the selected answers:

- Outcome:
- Non-goals:
- Architecture posture:
- Compatibility promises:
- Verification standard:
- Sequencing strategy:
- Concurrency posture:
- Risk posture:
- Deferred alternatives:

Ask the user to approve this ledger before writing scaffold files. If the user
corrects the ledger, update it and ask again. The approved ledger is the source
for `Alignment Summary`, `project-contract.md`, and `verification.md`.

## Approval Gate

Before writing scaffold files, get explicit approval of the decision ledger and
strategy. A useful approval prompt states:

```text
I will scaffold the roadmap around <recommended strategy>. It will treat
<scope> as in scope, keep <non-goals> out of scope, and use <verification> as
the completion signal. Approve this direction?
```

If the user says to proceed, that is approval. If the user provides corrections,
fold them into the alignment and ask again.

## Persistence Rules

After approval, persist alignment in the scaffolded control plane:

- `roadmap.md`: add `Alignment Summary` with approved thesis, success criteria,
  non-goals, chosen strategy, and deferred alternatives.
- `roadmap.md`: turn approved strategy into milestone and candidate-direction
  structure.
- `roadmap-view.json`: mirror milestone ids, direction ids, dependencies,
  statuses, and closeout anchors for runtime decisions.
- `project-contract.md`: record repo-wide invariants, compatibility promises,
  and architecture constraints that should outlive one roadmap family.
- `verification.md`: record baseline checks and alignment checks that prove the
  approved success criteria.

Do not create a separate brainstorming transcript file unless the user asks for
one. The scaffold checkpoint should remain the durable source of truth.

## Quality Bar

The approved alignment is good enough when:

- a human can see why this roadmap exists and what it will not do;
- the first milestone has an obvious completion signal;
- the verifier can connect checks back to the user's success criteria;
- the planner can later select lawful rounds without re-litigating the strategy;
- no placeholder strategy or generic project-management language remains.
