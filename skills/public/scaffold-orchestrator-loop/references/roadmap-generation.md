# Roadmap Generation

Build the initial roadmap from the goal plus a fast repository survey.

## Inputs

- User goal
- Existing code, docs, tests, and config
- Current branch status
- Constraints discovered in the repo

## Process

1. Survey the repository before drafting tasks.
2. Identify the smallest meaningful milestones that move the goal forward.
3. Sequence milestones so each item leaves the repo in a coherent state.
4. Record dependencies only when they change ordering.
5. Keep later items coarse; make the next item concrete.

## Roadmap Rules

- Use an ordered list.
- Give each item one clear deliverable.
- Record status with `[pending]`, `[in-progress]`, or `[done]`.
- Include `Depends on:` and `Completion notes:` lines for each item.
- Avoid planning parallel rounds or speculative implementation detail far ahead.
- Prefer 3-7 initial items unless the goal is truly smaller.

## First Item Quality Bar

The first actionable roadmap item should be concrete enough that a guider can select it without guessing:

- clear outcome
- obvious verification target
- bounded scope

## Revision Rules

The roadmap is expected to change after accepted rounds. Keep it stable enough to guide execution, but lightweight enough that the guider can update it without rewriting the whole file.
