---
name: Operational Functional Testing
trigger: model_decision
description: Require operational acceptance tests for Medium+ features that change real product workflows.
---

# Operational Functional Testing

For Medium+ features, technical tests are necessary but not sufficient. If a change introduces or refactors a real business capability, define operational functional tests before considering it shipped.

## Required

- Use `/operational-test-plan` after `/test-plan` when a feature touches business workflows, pipelines, agents, operators/admins, webhooks, queues, crons, external integrations, state machines, or multi-step user journeys.
- Validate the real operating flow: actor input → system behavior → persisted state → visible outcome → side effects.
- Prefer stable boundaries: public endpoints, facades, workflow entrypoints, queue/webhook handlers, or user-facing UI flows.
- If no stable boundary exists, add a task to create a facade or test harness before shipping.
- Do not mark a Medium+ workflow feature as shipped until the blocking operational scenarios pass or the user explicitly accepts the risk.

## Distinction

- Unit tests prove functions.
- Integration tests prove modules connect.
- E2E UI tests prove screens and clicks.
- Operational functional tests prove the product workflow works in realistic use.

## Anti-Patterns

- Shipping because unit/integration tests passed when no realistic business flow was executed.
- Testing only the UI while ignoring state, side effects, or operator handoff.
- Mocking the system behavior the feature is supposed to prove.
- Closing issues with failed or unexecuted blocking operational scenarios.
