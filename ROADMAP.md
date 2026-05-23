# Roadmap

High-level direction for spec-plan-ship. Actionable items are tracked as [GitHub issues](https://github.com/diegoesolorzano/spec-plan-ship/issues).

## Current State (v1)

Four-file skill kit: `/feature-spec`, `/feature-plan`, `/tdd`, and the `feature-workflow` rule. Covers spec → plan → test → ship with adversarial plan review.

## In Progress

- **Test plan phase** — `/test-plan` skill that designs all test cases upfront before implementation, forcing edge-case thinking before code exists. ([#1](https://github.com/diegoesolorzano/spec-plan-ship/issues/1))
- **Parallel task execution** — Workflow rule update so independent plan tasks execute concurrently via subagents. ([#2](https://github.com/diegoesolorzano/spec-plan-ship/issues/2))

## Planned

- **Init skill improvements** — Update `/spec-plan-ship-init` to install the new `/test-plan` skill alongside existing ones.
- **Cross-model adversarial review** — Option to send plan/test reviews to a different model for diversity of opinion. ([#3](https://github.com/diegoesolorzano/spec-plan-ship/issues/3))
- **Plan task status tracking** — Mark tasks as done during implementation so the workflow knows where you left off across sessions. ([#4](https://github.com/diegoesolorzano/spec-plan-ship/issues/4))
- **Spec templates by domain** — Lightweight domain-specific spec templates (API, UI feature, migration, CLI tool) that `/feature-spec` selects automatically.

## Design Principles

- **No CLI tools, no dependencies** — Pure skill files that work in any Claude Code project.
- **Detail scales to complexity** — Simple features get lightweight artifacts, complex features get comprehensive ones.
- **Real over mocked** — Tests run against real services by default. Mocks are a last resort.
- **Adversarial by default** — Every plan and test plan gets reviewed before the user sees it.
