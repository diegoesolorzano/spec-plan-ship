# Test Plan Scoring Rubric

Scoring guide for evaluating test artifact quality across three variants:
- **Variant A (inline):** TDD ad-hoc — tests designed per-task during implementation
- **Variant B (matrix):** Test Matrix embedded in the feature plan
- **Variant C (full):** Separate `/test-plan` skill output

## Dimensions

### 1. Acceptance Criteria Coverage (1-5)

Does every acceptance criterion from the spec have at least one corresponding test case?

| Score | Criteria |
|-------|----------|
| 1 | Fewer than half of acceptance criteria have tests |
| 2 | Most acceptance criteria covered, but 2+ are missed |
| 3 | All acceptance criteria have at least one test, some are weak |
| 4 | All acceptance criteria have specific, falsifiable tests |
| 5 | All acceptance criteria covered with multiple test cases (happy + edge + error per criterion) |

### 2. Edge Case Detection (1-5)

Does the test design surface edge cases beyond what the spec explicitly states?

| Score | Criteria |
|-------|----------|
| 1 | Only happy path tests |
| 2 | Happy path + 1-2 obvious error cases |
| 3 | Happy + error paths, misses non-obvious edges (concurrency, boundary values, null) |
| 4 | Good edge coverage including boundary values and error recovery |
| 5 | Catches non-obvious cases: race conditions, cascading failures, implicit invariants, state transitions |

### 3. Cross-Task Integration (1-5)

Do tests verify that outputs from one task correctly feed into downstream tasks?

| Score | Criteria |
|-------|----------|
| 1 | Each task tested in isolation, no integration awareness |
| 2 | Some integration awareness, but contract boundaries are untested |
| 3 | Integration points identified, tests exist but are vague |
| 4 | Key integration boundaries have specific tests with concrete assertions |
| 5 | Every contract boundary between tasks has a test verifying data shape, type, and behavior |

### 4. Test Quality (1-5)

Are the test cases well-designed — behavior-focused, falsifiable, independent?

| Score | Criteria |
|-------|----------|
| 1 | Tests describe implementation ("calls function X") not behavior |
| 2 | Mix of behavior and implementation tests |
| 3 | Tests describe behavior, but some are tautological or too vague to falsify |
| 4 | Tests are behavior-focused, falsifiable, and independent |
| 5 | Each test has specific Given/When/Then, would fail if the implementation is wrong, and can run in any order |

### 5. Infrastructure Awareness (1-5)

Does the test design account for practical test infrastructure needs?

| Score | Criteria |
|-------|----------|
| 1 | No mention of test setup, frameworks, or fixtures |
| 2 | Names the test framework, no fixture or setup detail |
| 3 | Test files named, framework identified, some fixture detail |
| 4 | Fixtures, factories, setup/teardown, and test isolation are addressed |
| 5 | Complete infrastructure: fixtures, factories, database strategy, cleanup hooks, CI considerations |

### 6. Actionability (1-5)

Can a cold agent write executable tests from this artifact alone?

| Score | Criteria |
|-------|----------|
| 1 | Vague descriptions — "test the API" — agent must design everything |
| 2 | Test file paths and general descriptions, agent must figure out specifics |
| 3 | Test cases named with rough assertions, agent fills in details |
| 4 | Test cases have Given/When/Then, specific assertions, agent writes mechanical code |
| 5 | Test cases are detailed enough that two agents would produce near-identical test files |

## Composite Score

All three variants use the same 6 dimensions (max 30):

```
score = (coverage + edge_cases + integration + quality + infrastructure + actionability) / 30
```

## Scoring Notes

- Score each variant independently — do not reference other variants while scoring
- Quote specific test cases from the artifact to justify scores
- For Variant A (inline), the "artifact" is the combined test output from all tasks treated independently
- For Variant B (matrix), the "artifact" is the Test Matrix section of the plan
- For Variant C (full), the "artifact" is the complete test plan document
