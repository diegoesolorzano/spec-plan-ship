---
name: Feature Workflow
trigger: model_decision
description: Orchestrates feature development flow from spec to commit
---

# Feature Development Workflow

When implementing a new feature or significant change (not a typo or 1-line fix):

1. **Spec first** — Use `/feature-spec` to define WHAT and WHY
2. **Plan second** — Use `/feature-plan` to define HOW and WHERE
3. **Implement with TDD** — For each plan task marked `Tests: Yes`, use `/tdd` to write tests first, then implement. For tasks without tests, implement directly.
4. **Review** — Review changes before committing
5. **Commit** — Create atomic, well-described commits

Skip to step 3 for trivial changes (config tweaks, copy edits, single-file fixes).

## Artifacts Location

| Artifact | Path | Created By |
|----------|------|------------|
| Feature specs | `docs/specs/{slug}.md` | `/feature-spec` |
| Implementation plans | `.claude/plans/{slug}-plan.md` | `/feature-plan` |
