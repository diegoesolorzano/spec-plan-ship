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

### Step 2.5: Classify the Tier

With the exploration done (never before — the criteria require knowing what the change
touches), classify the change using the decision table in `references/tier-policy.md`
(bundled with this skill): **Full / Standard / Lite**, plus the transversal modifiers
(`llm-evals` for any non-deterministic behavior change; `security-review` whenever a
security-gate trigger applies — in every tier). Promotion rule: in doubt, the higher tier;
a higher trigger discovered later promotes the change.

Then branch:

- **Lite** → do NOT produce a spec. Tell the user the change is Lite, name the verification
  its change type requires (tier-policy table), and hand off to direct implementation
  (`/feature-plan` lite mode via inline description is optional). Exit this skill.
- **Standard** → produce the spec with the required reduced section set ONLY: Problem
  Statement, Requirements, Acceptance Criteria, Sprint Goal — body ≤ 60 lines.
- **Full** → continue with the complete flow below.

Record the classification in the spec **after the contract header block and one blank
line** (outside the six normative fields):

```markdown
**Tier:** standard
**Modifiers:** llm-evals
```

### Step 3: Draft the Spec

The spec should be **as detailed as the feature requires**. A simple endpoint gets a short spec. A feature that involved evaluating multiple approaches, analyzing production data, or debating architecture gets a comprehensive spec that captures all of that context.

**If the feature is Medium+, read `references/agent-docs-standard.md` (bundled with this skill)** for the normative Agent Readiness templates and criteria (applicability matrix, AGENTS.md template, skill conformance, lightweight-rule criterion). Apply its **edit-before-create default**: extend/update the existing skill, AGENTS.md, or rule; create a new one only for a genuinely new subsystem, folder, or constraint.

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

## Agent Readiness (Medium+ only) — MANDATORY deliverable

For a Medium+ feature, the docs below are **part of the spec's scope**, not an afterthought:
list the applicable ones as concrete deliverables in `## Scope` and gate them in the Sprint
Goal's "Done when" with command-shaped evidence (file exists, `[ "$(readlink CLAUDE.md)" = "AGENTS.md" ]`, skill body <500 lines). A Medium+ feature that ships without its docs is **not done**.

Applicability matrix — deliverables are mandatory by default; every N/A requires an explicit 1-line justification (never a silent omission). Default: **update the existing artifact**; create a new one only for a genuinely new subsystem, folder, or constraint:
- **Skill**: always for Medium+ — extend the owning subsystem skill, or create a new one only for a distinct subsystem; N/A only if trivial or fully covered by an existing skill/rule (name which one).
- **AGENTS.md** (+ CLAUDE.md symlink): required iff the feature creates or modifies a module/facade folder; keep it short (purpose, public surface, non-obvious invariants, gotchas, pointer to the owning skill — never file inventories).
- **Rule**: required iff the constraint can be violated on ANY turn (not merely "important"); if adding one, state why it passes the every-turn filter, give the 1-line signpost version (invariants + pointer to the skill), and name the skill that receives the detail — normative detail never lives in the rule.

### Human Documentation
- **README.md**: Does this feature add commands, env vars, setup steps, or concepts that a human contributor needs to know?

### Agent Documentation
- **Runbook** (`docs/runbooks/{id}.md`): if the feature has a repeatable operational procedure, a multi-surface invariant that drifts easily, or manual recovery steps — write it and reference it from the skill (progressive disclosure).

### Discoverability Check
Can a future agent — with zero context from this conversation — find and correctly use this feature by reading project docs alone (AGENTS.md → skill → runbook)? If not, what's missing?

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
