---
name: feature-spec
description: >
  Feature specification before coding. Captures WHAT, WHY, and key design
  decisions. Produces specs with problem statement, requirements, acceptance
  criteria, design decisions, schema/API/UI changes, risks, and deploy checklist.
  Detail scales to complexity — simple features get short specs, complex features
  get comprehensive specs that capture the full decision context.
  Use when: "new feature", "I want to add", "let's build", "feature spec",
  or before any non-trivial implementation.
  Saves spec to docs/specs/ for review and approval before planning.
allowed-tools: Read, Glob, Grep, Write, AskUserQuestion
---

# Feature Spec

You are a **Product Manager + Technical Lead**. Your job is to define WHAT to build, WHY, and the key decisions that shape implementation.

## Procedure

### Step 1: Understand the Request

If the request is vague or incomplete, use Socratic questioning (max 3 rounds):
- Who benefits from this? Who is the user?
- What specific problem does this solve?
- What does "done" look like? How will we know it works?
- Are there edge cases or constraints to consider?

Do NOT proceed until you have a clear understanding of the feature.

### Step 2: Explore Impacted Systems

Use Glob, Grep, and Read to scan the codebase and identify:
- Database schemas or migrations affected
- API routes or endpoints involved
- UI components and pages
- State management (stores, context, etc.)
- Shared libraries and utilities
- Existing project rules (check `.claude/rules/` if present)

Reference specific file paths in the spec.

### Step 3: Draft the Spec

The spec should be **as detailed as the feature requires**. A simple endpoint gets a short spec. A feature that involved evaluating multiple approaches, analyzing production data, or debating architecture gets a comprehensive spec that captures all of that context.

Use the template below. Include all sections that apply — skip sections that don't.

The spec MUST open with the cross-link header block (after the optional `# ` H1), one field per line in this exact order: **Feature ID** (the `{id}`), **Repo** (the target repo's directory name), **Issue** (`{repo}#{n}` or `none`), **Upstream** (`none` for a spec — it has no upstream), **Date** (`YYYY-MM-DD`), **Status** (one of `Draft`, `Approved`, `Planned`, `Tested`, `Implementing`, `InReview`, `Shipped` — a fresh spec is `Draft`).

```markdown
# Feature: {Title}

**Feature ID:** {id}
**Repo:** {target repo directory name}
**Issue:** {repo}#{n} | none
**Upstream:** none
**Date:** YYYY-MM-DD
**Status:** Draft

## Problem Statement

1-3 sentences: What problem does this solve? Who is affected? What's the motivation?

## Background (only when it adds value)

Context that someone needs to understand WHY this approach was chosen:
- Data or evidence that informed the decision (production numbers, cost analysis)
- Concrete before/after examples showing what changes
- Key alternative considered and why it was rejected (one line, not a full analysis)

Skip this section for straightforward features where the "why" is obvious.

## Requirements

- [ ] FR-1: {Functional requirement}
- [ ] FR-2: {Functional requirement}
- [ ] FR-3: ...

## Acceptance Criteria

- **Given** {precondition}, **When** {action}, **Then** {expected result}
- **Given** ..., **When** ..., **Then** ...

## Sprint Goal

> {1-2 sentence goal statement — the outcome this sprint delivers.}

### Done when

Make every item **machine-verifiable** so the goal is precise enough to check — and to automate later: a shell command expected to exit 0, a named test/suite that must pass, a file/path that must exist, or a grep/pattern that must match. State the evidence inline. Avoid vague phrasings ("no regressions", "works well") unless bound to a concrete command (e.g. the full test suite exits 0).

- [ ] {Acceptance criterion} — evidence: `{command}` exits 0 / `{test}` passes / `{path}` exists
- [ ] ...
- [ ] All tests pass — evidence: `{repo test command}` exits 0

## Scope

Group deliverables by area. Be specific about file paths and what changes:

### [Area Name]
- [ ] `path/to/file.ts` — What changes and why

### Testing
- [ ] Unit tests: what to test, edge cases
- [ ] E2E tests: full flow verification
- [ ] Integration/evals: production-level validation (if applicable)

## Design Decisions

Key decisions that affect implementation. For each:
- **[Decision]**: [choice] because [reason]

Include the reasoning so future readers understand not just WHAT but WHY.

## Schema Changes (if any)

```sql
-- Migration: NNN_description.sql
ALTER TABLE ...
-- Rollback:
-- ...
```

## API Changes (if any)

- Endpoint: METHOD /path
- Request: { ... }
- Response: { ... }
- Auth: how it's authenticated

## UI Changes (if any)

Describe what the user sees, state transitions, loading/error states.
Not mockups — words describing the experience.

## Out of Scope

What this feature explicitly does NOT include and why.

## Agent Readiness (Medium+ only)

Answer each question. If the answer is "no" or "N/A", skip it.

### Human Documentation
- **README.md**: Does this feature add commands, env vars, setup steps, or concepts that a human contributor needs to know?

### Agent Documentation
- **AGENTS.md** (+ CLAUDE.md symlink): What does a cold agent — with zero prior context — need to discover and correctly use this feature? Keep root minimal; point to nested docs for detail (progressive disclosure).

### Rules
- Does this feature introduce constraints an agent must always respect? (e.g., "never call X without Y", "all routes in this module must Z")
- Does it introduce path-specific conventions that only apply when touching certain files?

### Skills
- Does this feature have a repeatable workflow that an agent or user should invoke on demand?
- Does it introduce process-specific guidance that is too heavy to always load but must apply conditionally? (e.g., domain-specific build steps, deploy procedures, conditional validation logic)

### Discoverability Check
Can a future agent — with zero context from this conversation — find and correctly use this feature by reading project docs alone? If not, what's missing?

## Risks

For each risk:
- **Risk**: what could go wrong
- **Mitigation**: how we handle it
- **Rollback**: how to revert if needed

## Deploy Checklist

- [ ] Database: migrations (list each)
- [ ] Tests: what must pass before deploy
- [ ] Deploy: services to deploy in order
- [ ] Verification: post-deploy checks
- [ ] Monitoring: what to watch after deploy
- [ ] Pending: non-blocking follow-ups

## Open Questions

Unresolved decisions, if any.
```

### Step 3.5: Define the Sprint Goal

Write the `## Sprint Goal` section (placed right after `## Acceptance Criteria`):

1. **Goal statement** — synthesize the Problem Statement into a 1-2 sentence outcome: what is true when this sprint is done?
2. **Done when** — turn each acceptance criterion from `## Acceptance Criteria` into a checkbox whose completion is **machine-verifiable**: a shell command expected to exit 0, a named test/suite that must pass, a file/path that must exist, or a grep/pattern that must match. Name the evidence inline. Make the criteria detailed and precise enough that a machine could later check them without judgment. Add "All tests pass" (bound to the repo's test command); for Medium+ features add "Agent docs updated (rules/skills/CLAUDE.md)" if applicable.

Invest in this section — a detailed, verifiable Sprint Goal is what makes the feature automatable later. The Sprint Goal lives in the spec (not the plan); `/feature-plan` references it rather than re-authoring it. The human still decides how to proceed from there — case by case.

### Step 4: Present for Approval

Show the complete spec to the user. Ask:
- "Does this capture everything? Any requirements missing?"
- "Anything that should be out of scope?"

### Step 5: Save the Spec

Once approved, save to `docs/specs/{id}.md`.

Update the status to `Approved`.

#### Handoff Brief (MANDATORY)

After saving, present a compact summary suitable for external audit or async review:

```
## Spec Brief

**Feature:** {title}
**Spec:** `docs/specs/{id}.md`
**Problem:** {1 sentence — who is affected and what's broken/missing}
**Key decisions:**
- {decision}: {choice} — because {reason}
- ...
**In scope:** {bullet list of what ships}
**Out of scope:** {bullet list of what doesn't}
**Top risks:** {1-2 most relevant risks with mitigations}

## Sprint Goal
> {goal statement quoted verbatim from the spec's ## Sprint Goal}
### Done when
- [ ] {criterion}
- [ ] ...

**Next:** `/feature-plan docs/specs/{id}.md`
```

This is a reformatting of information already in the spec — not new content. Its purpose is to give reviewers a self-contained snapshot without opening the file. **Always display the `## Sprint Goal` verbatim here** — it is the last thing shown before planning, so the human sees the target before deciding how to proceed.

Suggest next step: "Ready to plan implementation? Use `/feature-plan docs/specs/{id}.md`"

## Guidelines

- Focus on WHAT, WHY, and key HOW decisions. Leave granular implementation details for `/feature-plan`.
- Reference existing project rules from `.claude/rules/` when they constrain the feature.
- Match detail to complexity — don't over-document simple things, don't under-document complex ones.
- If alternatives were debated in conversation, capture the decision context in the spec.
- Do NOT write implementation code in the spec.
- Do NOT include UI mockups or wireframes (use words).
- Do NOT omit context from the conversation — if options were evaluated, capture the outcome.
- Do NOT leave decisions unexplained — always include the "because".
