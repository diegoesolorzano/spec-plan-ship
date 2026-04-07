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
model: opus
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

Use the template below. Include all sections that apply — skip sections that don't:

```markdown
# Feature: {Title}

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

### Step 4: Present for Approval

Show the complete spec to the user. Ask:
- "Does this capture everything? Any requirements missing?"
- "Anything that should be out of scope?"

### Step 5: Save the Spec

Once approved, save to `docs/specs/{YYYY-MM-DD}-{feature-slug}.md`.

Update the status to `Approved`.

Suggest next step: "Ready to plan implementation? Use `/feature-plan docs/specs/{feature-slug}.md`"

## Guidelines

- Focus on WHAT, WHY, and key HOW decisions. Leave granular implementation details for `/feature-plan`.
- Reference existing project rules from `.claude/rules/` when they constrain the feature.
- Match detail to complexity — don't over-document simple things, don't under-document complex ones.
- If alternatives were debated in conversation, capture the decision context in the spec.
- Do NOT write implementation code in the spec.
- Do NOT include UI mockups or wireframes (use words).
- Do NOT omit context from the conversation — if options were evaluated, capture the outcome.
- Do NOT leave decisions unexplained — always include the "because".
