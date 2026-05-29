---
name: feature-plan
description: >
  Granular implementation plan with ordered tasks and typed architectural
  anchors. Acts as Software Architect to define HOW, WHERE, and in what ORDER.
  Reads spec from docs/specs/ or accepts inline description. Produces tasks of
  2-5 min each with exact file paths, typed signatures (interfaces, function
  contracts), specific instructions, verification steps, and test requirements.
  References existing patterns in the codebase. Includes adversarial review
  via subagent before presenting. Use after /feature-spec or directly for
  small-medium changes.
argument-hint: [spec-file-path]
allowed-tools: Read, Glob, Grep, Write, AskUserQuestion, Agent, TaskCreate, TaskUpdate
model: opus
---

# Feature Plan

You are a **Software Architect**. Your job is to define HOW to build it, WHERE the code goes, and in what ORDER — anchoring architectural decisions with typed signatures.

## Procedure

### Step 1: Load the Spec

- If the user provided a file path as argument, read it.
- If no argument, ask: "Do you have a spec file? Or describe the change you want to implement."
- If a spec exists in `docs/specs/`, confirm it's the right one.

### Step 2: Assess Complexity

Determine if this change is **trivial** or **non-trivial**:

- **Trivial** (1-3 tasks, single layer, obvious pattern): Use lite mode — skip Shared Types, skip "Produces:" blocks, use the simple task format.
- **Non-trivial** (4+ tasks, multiple layers, architectural decisions): Use full mode — include Shared Types and typed "Produces:" blocks on every task.

When in doubt, use full mode.

### Step 3: Deep Codebase Exploration

Read the relevant files to understand existing patterns:

1. **Project rules** — Check `.claude/rules/` for domain-specific rules that apply to this feature.

2. **Existing implementations** — Read similar existing code to follow established conventions:
   - API routes (follow existing structure)
   - Components (follow naming and organization)
   - Libraries and utilities (follow patterns, reuse before creating new)
   - State management (follow existing conventions)

3. **Project documentation** — Check for architecture docs, READMEs, or learned patterns.

4. **Type system** — Identify relevant existing types, interfaces, and module boundaries that the new code must integrate with.

### Step 4: Define the Sprint Goal

Write the `## Sprint Goal` section for the plan:

1. **Goal statement** — Synthesize the spec's Problem Statement into a 1-2 sentence outcome. Write it as a completion criterion: what is true when this sprint is done? An autonomous agent reading only this sentence should understand what it's building toward.
2. **Done when** — Pull each acceptance criterion from the spec's `## Acceptance Criteria` section as a checkbox item. Add "All tests pass" and "No regressions in existing functionality" as final items. For Medium+ features, also add "Agent docs updated (rules/skills/CLAUDE.md)" if applicable.

Keep it concise — the goal is a reference target, not a restatement of the full spec.

### Step 5: Design the Task Breakdown

Break the implementation into tasks. Each task should be:
- **2-5 minutes** of focused work
- **Atomic** — one clear thing to do
- **Ordered** by dependencies (database first, then API, then UI)
- **Verifiable** — includes how to confirm it works
- **Architecturally anchored** — includes the typed shape of what it produces (full mode)

After defining all tasks, add a **Test Matrix** section (full mode only):
1. Map every acceptance criterion from the spec to at least one test case
2. Identify shared test infrastructure (fixtures, factories, setup/teardown)
3. For each task marked `Tests: Yes`, design test cases with Given/When/Then
4. Include the contract-under-test signature (from the task's Produces block)
5. For cross-layer features (DB + API + Store + UI), add 2-3 E2E flow descriptions
6. Independently identify pure functions that should be tested even if the task is marked `Tests: No`

#### Full Mode Template (default)

```markdown
# Plan: {Feature Title}

**Feature ID:** {id}
**Repo:** {target repo directory name}
**Issue:** {repo}#{n} | none
**Upstream:** docs/specs/{id}.md
**Date:** YYYY-MM-DD
**Status:** Planned

## Overview

1-2 sentences: architectural approach and key decisions.

## Sprint Goal

> {1-2 sentence goal statement — the outcome this sprint delivers, written so an autonomous agent knows when to stop.}

### Done when

- [ ] {Acceptance criterion from spec — Given/When/Then or declarative}
- [ ] {Acceptance criterion}
- [ ] ...
- [ ] All tests pass
- [ ] No regressions in existing functionality

## Shared Types

Types/interfaces that multiple tasks reference. Define them here once.

```typescript
// Cross-cutting contracts for this feature
interface ExampleInput { ... }
interface ExampleOutput { ... }
```

## Tasks

### Task 1: {Title}
- **Files:** `path/to/file.ts` (modify), `path/to/new-file.ts` (create)
- **Produces:**
  ```typescript
  // The typed shape this task must create/export
  export async function doThing(input: ExampleInput): Promise<ExampleOutput>;
  ```
- **Do:** Natural language intent — what this function/module should accomplish, edge cases to handle, patterns to follow. For visual/creative code, include specific formulas, values, and curves.
- **Integrates with:** Which existing modules/functions this connects to, and how.
- **Verify:** How to confirm this task is done correctly.
- **Tests:** Yes/No. If yes, what behaviors to test.
- **Depends on:** None | Task N

### Task 2: {Title}
...

## Test Matrix

### Shared Test Infrastructure
- **Framework:** {detected test framework}
- **Fixtures:** {shared test data across tasks}
- **Factories:** {builder functions for complex test objects}

### Acceptance Criteria → Test Mapping

| AC | Test Location | Case |
|----|--------------|------|
| Given X, When Y, Then Z | `path/to/test.ts` | case name |

### Per-Task Test Cases

#### Task N: {Title}
**File:** `path/to/file.test.ts`
**Type:** Unit | Integration | E2E
**Contract under test:**
```typescript
// The signature being tested (from Produces block)
export async function doThing(input: ExampleInput): Promise<ExampleOutput>;
```

| # | Case | Given | When | Then | Type |
|---|------|-------|------|------|------|
| 1 | happy path | ... | ... | ... | unit |
| 2 | edge case | ... | ... | ... | unit |
| 3 | error path | ... | ... | ... | unit |

(repeat for each task marked Tests: Yes)

### E2E Flows (for cross-layer features)

1. {User journey} → assert {result}
2. {Step} → assert {result}

## Patterns to Reuse

- Reference existing code with file paths
- Reference rules from `.claude/rules/`

## Risks

- Potential issues and how to mitigate them
```

#### Lite Mode Template (trivial changes only)

```markdown
# Plan: {Feature Title}

**Feature ID:** {id}
**Repo:** {target repo directory name}
**Issue:** {repo}#{n} | none
**Upstream:** docs/specs/{id}.md | none
**Date:** YYYY-MM-DD
**Status:** Planned

## Sprint Goal

> {1-sentence goal — what "done" looks like.}

### Done when

- [ ] {Key acceptance criterion}
- [ ] All tests pass

## Tasks

### Task 1: {Title}
- **Files:** `path/to/file.ts` (modify)
- **Do:** Specific instruction.
- **Verify:** How to confirm.
- **Depends on:** None
```

### Step 6: Adversarial Plan Review

**Before presenting the plan to the user**, save the draft to `.claude/plans/{id}-plan.md` and launch a subagent to review it adversarially.

Use the Agent tool with `subagent_type: "Plan"` and the following prompt:

```
You are a Senior Staff Engineer reviewing an implementation plan. Your job is to find flaws, gaps, and improvements. Be adversarial — assume the plan has problems.

Read these files:
1. The plan draft: .claude/plans/{id}-plan.md
2. The feature spec: docs/specs/{id}.md
3. The project instructions (CLAUDE.md or AGENTS.md if they exist)

Then critique the plan on these dimensions:

SIGNATURES & TYPES:
- Are the signatures complete enough to anchor implementation?
- Are there type mismatches or impossible contracts?
- Are return types specific enough (avoid `any`, `unknown` without justification)?
- Do the signatures compose correctly across tasks?

COMPLETENESS:
- Are all spec requirements covered by at least one task?
- Are there missing tasks (e.g., error handling, edge cases, migration rollback)?
- Are verification steps actually verifiable?

ARCHITECTURE:
- Are there better approaches than what was chosen?
- Does the plan follow existing project patterns or introduce unnecessary novelty?
- Are there coupling risks or over-engineering?

ORDERING & DEPENDENCIES:
- Are dependencies correctly identified?
- Could tasks be parallelized better?
- Is the critical path optimal?

RISKS:
- Are there unidentified risks?
- Are the mitigations realistic?
- What could go wrong that isn't mentioned?

TEST COVERAGE:
- Does every acceptance criterion have at least one test case in the Test Matrix?
- Are there pure functions marked `Tests: No` that should have unit tests?
- For cross-layer features, are there E2E flows testing the integration boundaries?
- Are contract-under-test signatures present for each testable task?

SCOPE:
- Is anything over-engineered for what the spec asks?
- Are there tasks that could be deferred without blocking the feature?

Output a structured review with:
## Findings
- [CRITICAL] Issues that must be fixed before implementing
- [WARNING] Issues worth addressing but not blockers
- [SUGGESTION] Improvements that could make the plan better

## Missing Tasks
Any tasks that should be added.

## Recommended Changes
Specific modifications to existing tasks.

Do NOT rewrite the plan. Only provide the review findings.
```

After receiving the subagent's review:
1. Incorporate all CRITICAL findings (mandatory)
2. Incorporate WARNING findings where they improve the plan
3. Consider SUGGESTION findings but don't over-engineer
4. Update the draft in `.claude/plans/{id}-plan.md`
5. Add a `## Review Notes` section at the end of the plan summarizing what changed

### Step 7: Present for Approval

Show the complete plan to the user. Mention that it was reviewed and improved by the adversarial agent. Ask:
- "Does this approach make sense?"
- "Want to adjust the scope, task order, or any signatures?"

### Step 8: Save the Plan

Once approved, update status in `.claude/plans/{id}-plan.md` if needed.

#### Next Step Guidance (MANDATORY)

Determine what comes next based on the feature-workflow rule. **Never suggest jumping to implementation directly.**

- **If the plan is full mode (non-trivial / Medium+ complexity):**
  Suggest: "Next step: `/test-plan .claude/plans/{id}-plan.md` to expand the Test Matrix into a full test suite before implementing."

- **If the plan is lite mode (trivial / Low complexity):**
  Display the Sprint Goal with the plan path and method as reference:
  ```
  **Plan:** `.claude/plans/{id}-plan.md`
  **Method:** `/tdd` (RED → GREEN → REFACTOR for each task)

  ## Sprint Goal
  > {quoted from plan}
  ```
  Then suggest: "Ready to implement? I'll invoke `/tdd` for each task — writing the failing test first, then implementing."

  **IMPORTANT:** Implementation MUST use the `/tdd` skill regardless of how the user phrases the request ("implement", "build it", "go ahead", etc.). Skipping `/tdd` violates the feature-workflow.

The feature-workflow sequence is: spec → plan → **test-plan (Medium+)** → **Sprint Goal display** → implement with /tdd. Skipping test-plan for Medium+ features violates the workflow.

#### Task Tracking (full mode only — MANDATORY)

After the plan is approved, create trackable tasks for the implementation phase:

1. For each task in the plan, call `TaskCreate` with:
   - **subject:** "Task N: {Task Title}" (from the plan)
   - **description:** The "Do" block content from that task
   - **activeForm:** "Implementing Task N: {Title}"

2. After all tasks are created, set up dependencies with `TaskUpdate`:
   - For each task that has `Depends on: Task M`, call `TaskUpdate` with `addBlockedBy: ["{task-M-id}"]`

3. Do NOT mark any task as `in_progress` — that happens when `/tdd` starts each task during implementation.

This gives the implementing agent (or parallel subagents) a live progress tracker. Each subagent should `TaskUpdate` its assigned task to `in_progress` when starting and `completed` when done.

## Guidelines

### Signature Quality

- **Signatures must be valid TypeScript** (or the project's language) — not pseudo-code
- **Include parameter types and return types** — never leave them implicit
- **Use the project's existing type conventions** — if they use `Result<T, E>` patterns, use those
- **Generics over `any`** — constrain types as tightly as reasonable
- **Only export what's needed** — internal helpers don't need signatures in the plan

### What Goes in "Produces" vs "Do"

| In "Produces" (code) | In "Do" (prose) |
|---|---|
| Function signatures | Implementation approach |
| Interface/type definitions | Edge case handling strategy |
| Export shape | Error handling details |
| Key data structures | Performance considerations |
| Database schema (SQL) | Migration rollback strategy |
| — | Math formulas, curves, specific values for visual/creative code |

### What Does NOT Belong in Signatures

- Function bodies (implementation)
- Variable assignments
- Control flow
- Comments explaining the implementation
- Test code

### Task Quality

- **File paths must be exact** — relative to project root
- **"Do" must be specific** — not "implement the API" but "create POST handler that receives `{ email }`, validates format, inserts into `subscribers` table, returns 201"
- **"Verify" must be actionable** — not "it works" but "run `build` — no errors" or "POST to `/api/xxx` with `{ email: 'test@test.com' }` returns 201"
- **For visual/creative code** — include exact values (colors, curves, timing) in "Do", not just type annotations

### Task Ordering

Standard order for full-stack features:
1. Shared types/interfaces (the contracts)
2. Database migration (schema changes)
3. Library modules (business logic)
4. API routes (server endpoints)
5. State management changes (if needed)
6. UI components (client-side)
7. Integration/wiring (connecting everything)

### Agent Documentation Task

Every plan for a Medium+ feature MUST include a task (typically second-to-last, before deploy/close) for agent documentation:

- **Do:** Check the spec's "Agent Readiness" section. Create or update rules, skills, and CLAUDE.md/AGENTS.md entries as specified. If the spec has no Agent Readiness section, evaluate independently: does a future agent need any documentation to discover, use, or extend this feature?
- **Verify:** A cold agent reading only project docs can find and correctly use this feature.

### Reuse Over Reinvention

- Before proposing new utilities, check if similar ones exist
- Before proposing new components, check existing UI primitives
- Before proposing new patterns, check project rules and conventions
- Always reference the existing code you found as the pattern to follow

### Scope Control

- If the plan exceeds 10 tasks, consider splitting into phases
- Flag any task that feels risky or uncertain in the Risks section
- If you discover the feature is bigger than the spec suggested, flag it before proceeding
