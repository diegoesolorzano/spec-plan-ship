---
name: Documentation Scope
trigger: model_decision
description: Clarify what "document this" means across human docs, agent docs, rules, skills, and future work.
---

# Documentation Scope

When the user says to document a feature, bug, decision, behavior, or future option, do not assume a single README note is enough. Choose the documentation surface by who needs the information later.

## Required Triage

- Human contributor setup or usage → update `README.md` or domain docs.
- Architecture, data flow, or operating model → update `docs/architecture/` or equivalent project docs.
- Manual operations, deploys, credentials, dashboards, or incident steps → update `docs/runbooks/` or equivalent runbook docs.
- Agent-discoverable project map → update `AGENTS.md` with a brief pointer, not a full duplicate.
- Always-on constraint future agents must obey → create/update a rule.
- Conditional or detailed workflow with progressive disclosure → create/update a skill.
- Future option or deferred approach → record it in the spec, roadmap, or issue tracker.
- Historical change summary → changelog or aiCommits, if the project uses one.

## Rule of Thumb

- If future agents must behave differently, document it in `AGENTS.md`, rules, or skills.
- If future humans must operate or configure something, document it in README/docs/runbooks.
- If it is only history, use changelog/aiCommits; do not make changelog the source of truth.

## Anti-Patterns

- Leaving decisions only in chat.
- Saying "documented" when only a commit message or changelog exists.
- Adding a long always-on rule when a skill would be better progressive disclosure.
- Updating root `AGENTS.md` with details that belong in focused rules, skills, or docs.
