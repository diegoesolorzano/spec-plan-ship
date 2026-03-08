---
name: Feature Workflow
trigger: model_decision
description: Orchestrates feature development flow from spec to commit
---

# Feature Development Workflow

When implementing a new feature or significant change (not a typo or 1-line fix):

1. **Spec first** — Use `/feature-spec` to define WHAT and WHY
2. **Plan second** — Use `/feature-plan` to define HOW and WHERE
3. **Implement** — Execute tasks from the plan in order
4. **Test** — Write tests for tasks that require them
5. **Review** — Review changes before committing
6. **Commit** — Create atomic, well-described commits

Skip to step 3 for trivial changes (config tweaks, copy edits, single-file fixes).

## Artifacts Location

| Artifact | Path | Created By |
|----------|------|------------|
| Feature specs | `docs/specs/{slug}.md` | `/feature-spec` |
| Implementation plans | `.claude/plans/{slug}-plan.md` | `/feature-plan` |
