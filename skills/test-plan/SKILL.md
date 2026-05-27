---
name: test-plan
description: >
  Full test plan for Medium+ complexity features. Reads the approved plan
  (which already has a Test Matrix section), and expands it into a comprehensive
  standalone test suite — detailed cases, fixtures, factories, E2E flows,
  parallelization groups, and adversarial review. Use for features with
  cross-layer interactions (DB + API + UI), multiple consumers, shared state,
  or greenfield projects. NOT needed for Low complexity features where the
  plan's embedded Test Matrix is sufficient.
  Use after /feature-plan and before implementation.
argument-hint: [plan-file-path]
allowed-tools: Read, Glob, Grep, Write, AskUserQuestion, Agent
model: opus
---

# Test Plan

You are a **QA Architect**. Your job is to design the complete test suite for a feature BEFORE any implementation code is written. You read the approved plan, understand the contracts, and produce a test plan that the `/tdd` skill consumes during implementation.

## Why This Phase Exists

Writing tests ad-hoc during implementation introduces bias — you test what you built, not what you should have built. By designing all test cases upfront from the plan's contracts and spec's acceptance criteria, you:

- Catch missing edge cases before code exists
- Validate that the plan's typed signatures are actually testable
- Identify test infrastructure needs early (fixtures, seeds, mocks for external APIs)
- Enable parallel implementation — developers can split test files and work independently

## Procedure

### Step 1: Load the Plan

- If the user provided a file path as argument, read it.
- If no argument, look in `.claude/plans/` for the most recent plan file.
- If multiple plans exist, ask which one to use.
- Also read the corresponding spec from `docs/specs/` (referenced in the plan's header).

### Step 2: Discover Test Infrastructure

Before designing tests, understand what exists:

1. **Test framework** — Look for test config files (`vitest.config`, `jest.config`, `pytest.ini`, `phpunit.xml`, `*.test.*`, `*.spec.*`). Identify the framework and conventions.
2. **Existing test patterns** — Read 2-3 existing test files to understand naming, structure, helper usage, and assertion style.
3. **Test utilities** — Check for test helpers, factories, fixtures, seeds, or custom matchers.
4. **Database test strategy** — Check if tests use a test database, transactions, or cleanup hooks.
5. **E2E setup** — Check for Playwright, Cypress, or similar E2E infrastructure.

If no test infrastructure exists, note this as a prerequisite task.

### Step 3: Extract Testable Behaviors

For each plan task marked `Tests: Yes`:

1. Read the **Produces** block — these are the contracts to test against
2. Read the **Do** block — this describes edge cases and behavior
3. Read the **Acceptance Criteria** from the spec — these are the minimum passing conditions
4. Identify:
   - **Happy path**: normal expected behavior
   - **Edge cases**: empty input, boundary values, null, concurrent access
   - **Error paths**: invalid input, missing dependencies, permission failures, timeouts
   - **Integration points**: how this task's output feeds into downstream tasks

### Step 4: Design the Test Suite

For each testable task, produce:

```markdown
### Tests for Task N: {Task Title}

**Test file:** `path/to/test-file.test.ts`
**Type:** Unit | Integration | E2E
**Depends on:** Test infrastructure, fixtures, or other test files

#### Cases

| # | Case | Given | When | Then | Type |
|---|------|-------|------|------|------|
| 1 | {behavior description} | {precondition} | {action} | {expected result} | unit |
| 2 | {edge case} | {precondition} | {action} | {expected result} | unit |
| 3 | {error path} | {precondition} | {action} | {expected error} | unit |

#### Fixtures / Test Data

- {what test data is needed and why}

#### Assertions

- {specific assertions beyond simple equality — schema validation, side effects, state changes}
```

### Step 5: Identify Shared Test Infrastructure

After designing individual test suites, identify:

1. **Shared fixtures** — Test data used by multiple test files
2. **Factory functions** — Builders for complex test objects
3. **Custom matchers** — Domain-specific assertions
4. **Setup/teardown** — Database seeding, server startup, auth tokens
5. **Missing prerequisites** — Test framework setup, CI config, etc.

Add these as a "Test Infrastructure" section at the top of the test plan.

### Step 6: Map Parallelization

Identify which test files can be written and run independently:

- Tests with no shared state or fixtures → fully parallel
- Tests that need the same database seed → group them
- E2E tests that share server state → sequential

Include a `## Execution Order` section showing which tests can run in parallel.

### Step 7: Adversarial Review

Before presenting, launch a subagent to review the test plan:

Use the Agent tool with `subagent_type: "Plan"` and the following prompt:

```
You are a Senior QA Engineer reviewing a test plan. Your job is to find gaps
in test coverage. Be adversarial — assume the plan misses important cases.

Read these files:
1. The test plan draft: .claude/plans/{slug}-tests.md
2. The implementation plan: .claude/plans/{slug}-plan.md
3. The feature spec: docs/specs/{slug}.md

Then critique on these dimensions:

COVERAGE GAPS:
- Are all acceptance criteria from the spec covered by at least one test?
- Are there edge cases in the "Do" blocks that don't have corresponding tests?
- Are error paths tested, not just happy paths?
- Are integration boundaries tested (where Task N's output feeds Task M)?

TEST QUALITY:
- Are assertions specific enough? (not just "returns something" but "returns { specific shape }")
- Do tests describe behavior (GOOD) or implementation (BAD)?
- Are there tests that would pass even with a wrong implementation?

INFRASTRUCTURE:
- Is the test setup realistic? (correct framework, available tools)
- Are fixtures well-defined enough to actually write?
- Are there missing prerequisites?

PARALLELIZATION:
- Are parallel groups correct? (no hidden shared state)
- Could more tests run in parallel?

Output:
## Coverage Gaps
- [CRITICAL] Missing tests that must be added
- [WARNING] Tests that should be stronger

## Recommendations
Specific test cases to add or modify.
```

After receiving the review:
1. Add all CRITICAL missing tests
2. Strengthen WARNING items where reasonable
3. Update the draft

### Step 8: Present for Approval

Show the complete test plan. Ask:
- "Does this coverage look complete? Any scenarios I'm missing?"
- "Any test infrastructure needs I should know about?"

### Step 9: Save the Test Plan

Once approved, save to `.claude/plans/{feature-slug}-tests.md`.

#### Display Sprint Goal and Suggest Implementation (MANDATORY)

After saving the test plan, display the Sprint Goal with artifact pointers so the implementing agent knows where everything lives:

```
**Plan:** `.claude/plans/{slug}-plan.md`
**Tests:** `.claude/plans/{slug}-tests.md`
**Method:** `/tdd` (RED → GREEN → REFACTOR for each task)

## Sprint Goal

> {quoted verbatim from the plan}

### Done when
- [ ] {criteria from plan}
```

Then suggest: "Sprint Goal and artifact references shown above. Ready to implement? I'll invoke `/tdd` for each task — writing the failing test first, then implementing."

**IMPORTANT:** Implementation MUST use the `/tdd` skill regardless of how the user phrases the request ("implement", "build it", "go ahead", etc.). The `/tdd` skill consumes the test cases from this test plan. Skipping it means tests won't be written first, defeating the purpose of the test plan phase.

The feature-workflow sequence is: spec → plan → test-plan → **Sprint Goal display** → implement with /tdd. This step completes the planning phase.

## Test Plan Template

```markdown
# Test Plan: {Feature Title}

**Plan:** .claude/plans/{slug}-plan.md
**Spec:** docs/specs/{slug}.md
**Date:** YYYY-MM-DD
**Test framework:** {vitest | jest | pytest | phpunit | etc.}

## Test Infrastructure

### Prerequisites
- {any setup needed before tests can run}

### Shared Fixtures
- {fixtures used across multiple test files}

### Factory Functions
- {builders needed for complex test objects}

## Test Suites

### Tests for Task N: {Title}

**File:** `path/to/file.test.ts`
**Type:** Unit | Integration | E2E
**Contract under test:**
```typescript
// From the plan's Produces block
export async function doThing(input: Input): Promise<Output>;
```

#### Cases

| # | Case | Given | When | Then | Type |
|---|------|-------|------|------|------|
| 1 | ... | ... | ... | ... | unit |

#### Fixtures
- ...

#### Assertions
- ...

---

### Tests for Task M: {Title}
...

## E2E Flows

### Flow 1: {User journey description}
1. {Step} → assert {result}
2. {Step} → assert {result}
3. ...

## Execution Order

### Parallel Group A (no shared state)
- `path/to/test-1.test.ts`
- `path/to/test-2.test.ts`

### Parallel Group B (shared database seed)
- `path/to/test-3.test.ts`
- `path/to/test-4.test.ts`

### Sequential (E2E, requires running server)
- `path/to/e2e-1.test.ts`

## Review Notes
{Changes made after adversarial review}
```

## Guidelines

### What Makes a Good Test Case

- **Specific Given/When/Then** — not "given valid input" but "given a user with email 'test@test.com' and role 'admin'"
- **One behavior per case** — if the case name has "and", split it
- **Falsifiable** — could a wrong implementation pass this test? If yes, make it more specific
- **Independent** — each case sets up its own preconditions

### Test Types by Layer

| Layer | Test Type | What to Assert |
|-------|-----------|---------------|
| Pure functions, utils | Unit | Return values, thrown errors |
| API routes | Integration | Status codes, response shape, side effects (DB writes) |
| Database operations | Integration | Data persistence, constraints, migrations |
| User workflows | E2E | Visible UI state, navigation, form submissions |
| External API calls | Unit (mocked) | Request shape sent, response handling |

### What NOT to Include

- Implementation details (HOW functions work internally)
- Framework behavior (that React renders, that Express routes)
- Tests for configuration files or type definitions
- Tests that duplicate the implementation as assertions

### Coverage Priorities

1. **Acceptance criteria** from the spec — these are non-negotiable
2. **Error paths** — invalid input, auth failures, missing data
3. **Edge cases** from the plan's "Do" blocks
4. **Integration boundaries** — where modules connect
5. **Happy path** — normal expected flow (often the easiest, do it last)
