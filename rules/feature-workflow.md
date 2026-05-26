---
name: Feature Workflow
trigger: model_decision
description: Orchestrates feature development flow from spec to ship
---

# Feature Development Workflow

When implementing a new feature or significant change (not a typo or 1-line fix):

1. **Spec first** — Use `/feature-spec` to define WHAT and WHY
2. **Plan second** — Use `/feature-plan` to define HOW and WHERE (includes a Test Matrix and Sprint Goal in full mode)
3. **Test plan (Medium+ only)** — Use `/test-plan` for features with cross-layer interactions, multiple consumers, or shared state. Expands the plan's Test Matrix into a comprehensive standalone test suite with E2E flows, fixtures, and parallelization groups. Skip for Low complexity features — the plan's embedded Test Matrix is sufficient.
4. **Sprint Goal** — Before implementing, display the Sprint Goal from the plan. This is the acceptance criteria the agent must satisfy autonomously. Show it after the last planning artifact: after `/test-plan` if one was created, otherwise after `/feature-plan`. Format: quote the `## Sprint Goal` section from the plan file. Optional but recommended — gives agents a clear stopping criterion.
5. **Implement with TDD** — For each plan task, use `/tdd` (which consumes the test plan or Test Matrix) to write tests first, then implement. Pure functions get tests even if the task is marked `Tests: No`.
6. **Review** — Review changes before committing
7. **Commit** — Create atomic, well-described commits

Skip to step 5 for trivial changes (config tweaks, copy edits, single-file fixes).

## Parallel Execution

During step 5, tasks that have no dependency conflicts (`Depends on: None` or all dependencies already completed) SHOULD be executed in parallel via subagents. The implementation agent should:

1. Read the plan's dependency graph
2. Identify all tasks whose dependencies are satisfied
3. Launch independent tasks concurrently using the Agent tool
4. Wait for completion, then identify the next batch of ready tasks
5. Repeat until all tasks are done

Each parallel subagent receives:
- The task description from the plan
- The corresponding test cases from the test plan
- Instruction to follow `/tdd` cycle (RED → GREEN → REFACTOR)

Tasks with unmet dependencies wait until their blockers complete.

## Artifacts Location

| Artifact | Path | Created By |
|----------|------|------------|
| Feature specs | `docs/specs/{slug}.md` | `/feature-spec` |
| Implementation plans | `.claude/plans/{slug}-plan.md` | `/feature-plan` |
| Test plans | `.claude/plans/{slug}-tests.md` | `/test-plan` |
