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

### Step 4: Reference the Sprint Goal

The `## Sprint Goal` is authored in the **spec** (`/feature-spec`), not here. The plan does **not** re-author it.

- Read the spec's `## Sprint Goal` and reference it by its upstream path; the display-before-implement step (below) sources the Sprint Goal from the spec.
- If the spec predates the Sprint Goal placement (goal still in an old plan) or has no `## Sprint Goal`, flag it and ask the user — do not silently invent criteria here.

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

_Quoted from the spec's `## Sprint Goal` (authored in `/feature-spec`, not here). Reproduce it verbatim as a reference target._

> {goal statement quoted from the spec}

### Done when

- [ ] {criterion quoted from the spec}
- [ ] ...

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
- **Risky:** yes *(optional — omit unless true; forces this task to run alone)*
- **Shared infra:** yes *(optional — omit unless true; the task touches a shared DB, port, or service)*

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

**The last task of every plan is the security review** (see "Security Review Task" below). It is
part of the template, not an optional extra — emit it after the agent-docs task, and never
renumber it away.

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

_Quoted from the spec's `## Sprint Goal` (authored in `/feature-spec`, not here)._

> {goal quoted from the spec}

### Done when

- [ ] {criterion quoted from the spec}
- [ ] All tests pass

## Tasks

### Task 1: {Title}
- **Files:** `path/to/file.ts` (modify)
- **Do:** Specific instruction.
- **Verify:** How to confirm.
- **Depends on:** None
```

Lite plans still carry the security-review task whenever the change touches a fetch, an API
route, a DB read/write, a permission, or a secret — a one-line `GRANT` is a lite change and was
exactly the incident that motivated the gate. Only a plan with no new data access omits it.

### Step 6: Emit `## Execution Waves` (when the plan qualifies)

Compute this **now** — before the adversarial review, before cross-model review, before approval — so
that the plan that gets reviewed and approved is the final one. Inserting it later mutates an
approved artifact.

Offer it only when the plan qualifies: **the realizable schedule must contain at least one wave of
concurrency ≥2.** Task count alone is not the test — twelve tasks in a chain, or a single shared
staging DB, collapse to sequential work anyway. Below that bar, skip the section entirely and say
sequential `/tdd` is the answer.

Derive the waves deterministically (the lite template never qualifies — trivial plans do not
parallelize):

1. **Normalize** each task's `Files:` to repo-relative POSIX paths. Compare them **case-insensitively**
   (macOS and Windows treat `Src/a.ts` and `src/a.ts` as one file).
2. **Mark unknown** — if *any* entry is a glob, a directory, an absolute path, a `..` path, a home-dir
   path, or the bullet is missing entirely, the task's file set is not trustworthy.
3. **Force serial** any task that is unknown, a migration, `Risky: yes`, `Shared infra: yes`, or that
   writes a machine-global surface (`~/.claude`, `~/.agents`) — the last are orchestrator-only.
4. **Build waves uncapped**, in task-id order: a task joins the current wave only when every task it
   depends on is already in an earlier wave AND its files are disjoint from every task already in
   this wave. Forced-serial tasks take a wave alone. **Do not group by "connected component"** — if A
   collides with B and B with C, but A and C do not collide, A and C may still share a wave.
5. **Split by cap last** — one shared staging DB caps a wave at 1, an E2E cost group at 2 — preserving
   order.

Emit the result, plus the cap and every constraint that produced it, and the orchestrator-only line:

```markdown
## Execution Waves

| Wave | Task ids | Why they fit |
|---|---|---|
| 1 | T1, T3 | disjoint files, no dependencies |
| 2 | T2 | rewrites the file T1 created |

Forced serial: {ids and why} · Cap: {n} · {every active bound, or "no constraint; full parallel"}
Orchestrator-only: merges, PRs, migrations, deploys, and any paid review of the root artifacts.
```

The section is **advisory** — deleting it restores sequential `/tdd`. If the suite-core
`/parallel-ship` skill is installed, it computes this same schedule deterministically (and executes
it); this checklist is what the free workflow uses on its own.

### Step 7: Adversarial Plan Review

**Before presenting the plan to the user**, save the draft to `docs/specs/{id}-plan.md` and launch a subagent to review it adversarially.

Use the Agent tool with `subagent_type: "Plan"` and the following prompt:

```
You are a Senior Staff Engineer reviewing an implementation plan. Your job is to find flaws, gaps, and improvements. Be adversarial — assume the plan has problems.

Read these files:
1. The plan draft: docs/specs/{id}-plan.md
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
4. Update the draft in `docs/specs/{id}-plan.md`
5. Add a `## Review Notes` section at the end of the plan summarizing what changed

### Step 8: Present for Approval

Show the complete plan to the user. Mention that it was reviewed and improved by the adversarial agent. Ask:
- "Does this approach make sense?"
- "Want to adjust the scope, task order, or any signatures?"

### Step 9: Save the Plan

Once approved, update status in `docs/specs/{id}-plan.md` if needed.

#### Next Step Guidance (MANDATORY)

Determine what comes next based on the feature-workflow rule. **Never suggest jumping to implementation directly.**

- **If the plan is full mode (non-trivial / Medium+ complexity):**
  Suggest: "Next step: `/test-plan docs/specs/{id}-plan.md` to expand the Test Matrix into a full test suite before implementing. If this feature changes a real product workflow, run `/operational-test-plan docs/specs/{id}-plan.md` after `/test-plan`."

- **If the plan is lite mode (trivial / Low complexity):**
  Display the Sprint Goal with the plan path and method as reference:
  ```
  **Plan:** `docs/specs/{id}-plan.md`
  **Method:** `/tdd` (RED → GREEN → REFACTOR for each task)

  ## Sprint Goal
  > {quoted from plan}
  ```
  Then suggest: "Ready to implement? I'll invoke `/tdd` for each task — writing the failing test first, then implementing."

  **IMPORTANT:** Implementation MUST use the `/tdd` skill regardless of how the user phrases the request ("implement", "build it", "go ahead", etc.). Skipping `/tdd` violates the feature-workflow.

- **If the plan carries an `## Execution Waves` section** (Step 6 found at least one wave of
  concurrency ≥2): mention it once — "This plan has parallelizable waves; `/parallel-ship` can run
  them in isolated worktrees, or `/tdd` sequentially as usual." Offer it; never assume it. Parallel
  execution costs a worktree and a dependency install per concurrent task, so it earns its keep only
  on genuinely wide plans.

The feature-workflow sequence is: spec → plan → **test-plan (Medium+)** → **operational-test-plan when workflow behavior changes** → **Sprint Goal display** → implement with /tdd. Skipping test-plan for Medium+ features violates the workflow. Skipping operational-test-plan for Medium+ workflow features violates the workflow unless the user explicitly accepts the risk.

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

Every plan for a Medium+ feature MUST include a task (typically second-to-last, before deploy/close) that materializes the **applicable** deliverables from the spec's "Agent Readiness" section — never a vague "update docs" task:

- **Do:** For each applicable deliverable, update the existing artifact by default (extend the owning skill / update the folder's AGENTS.md / update the existing rule); create a new one only for a genuinely new subsystem, folder, or constraint. For each deliverable the spec marks N/A, **quote its 1-line justification and its source line from the spec** in the task — an N/A without a quoted justification is a gap the plan must flag, not accept. If the spec has no Agent Readiness section, evaluate independently: does a future agent need any documentation to discover, use, or extend this feature?
- **Verify (command-shaped, per applicable deliverable):**
  - AGENTS.md: `test -f {dir}/AGENTS.md` and `[ "$(readlink {dir}/CLAUDE.md)" = "AGENTS.md" ]` both exit 0
  - Skill: `[ "$(wc -l < .claude/skills/{name}/SKILL.md)" -lt 500 ]` exits 0, and its frontmatter `name` matches the directory
  - Rule: the rule body stays 1-line invariants + a pointer to the owning skill (no normative detail)
  - Plus the judgment check: a cold agent reading only project docs can find and correctly use this feature.

### Security Review Task (FIXED — last task of every plan)

Every plan ends with a task that runs the `security-review` skill over the finished diff. It is
**mandatory** when the feature adds or modifies a fetch/HTTP call, an API route or webhook, a DB
read/write (query, RPC, migration, trigger, view, policy), a permission/grant/role/access flag,
or secret handling — which is nearly every plan that is not pure UI. Emit it as:

```markdown
### Task N: Revisión de seguridad final
- **Files:** none (read-only review)
- **Do:** Invoke the `security-review` skill over the full diff of this feature. Enumerate the
  surface this plan adds — {list the endpoints / RPCs / migrations / grants from the tasks
  above} — and run its seven checks against it. Highest priority: what the BROWSER can reach
  directly (privileged keys in client-reachable code; `SECURITY DEFINER` functions with EXECUTE
  granted to `anon`/`authenticated`; views without caller-scoped execution).
- **Verify:** Every check has a command and its output. Zero CRITICAL findings, or each one has
  a fix task added below and completed before merge.
- **Depends on:** all implementation tasks
- **Risky:** yes
```

Rules: it goes **last** (there is nothing to review before the code exists), it is **read-only**
(findings become new fix tasks, never silent edits inside the review task), and a plan that omits
it must state in `## Risks` why the change has no new data-access surface. Never mark it done
with "reviewed, looks fine" — the skill's report table is the deliverable.

### Reuse Over Reinvention

- Before proposing new utilities, check if similar ones exist
- Before proposing new components, check existing UI primitives
- Before proposing new patterns, check project rules and conventions
- Always reference the existing code you found as the pattern to follow

### Scope Control

- If the plan exceeds 10 tasks, consider splitting into phases
- Flag any task that feels risky or uncertain in the Risks section
- If you discover the feature is bigger than the spec suggested, flag it before proceeding
