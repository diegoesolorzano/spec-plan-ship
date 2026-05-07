# Scoring Rubric

Scoring guide for evaluating plan quality across variants (standard vs hybrid).

## Dimensions

### 1. Completeness (1-5)

Does the plan cover all requirements from the spec?

| Score | Criteria |
|-------|----------|
| 1 | Misses >2 requirements, no migration or test tasks |
| 2 | Misses 1-2 requirements, gaps in non-functional concerns |
| 3 | All functional requirements covered, minor gaps (error handling, cleanup) |
| 4 | All requirements covered, includes error handling and edge cases |
| 5 | All requirements plus defensive tasks (rollback, validation, cleanup) |

### 2. Specificity (1-5)

Can an agent implement each task without asking clarifying questions?

| Score | Criteria |
|-------|----------|
| 1 | Tasks are vague ("implement the feature", "add the API") |
| 2 | Tasks name files but instructions are generic |
| 3 | Tasks have file paths and specific instructions, some ambiguity remains |
| 4 | Each task is unambiguous — one reasonable interpretation |
| 5 | Tasks are so precise that two agents would produce near-identical implementations |

### 3. Architecture (1-5)

Does the plan follow existing patterns and order dependencies correctly?

| Score | Criteria |
|-------|----------|
| 1 | Ignores existing patterns, wrong dependency order |
| 2 | References some existing code, order has issues |
| 3 | Follows patterns, correct order, minor coupling concerns |
| 4 | Clean architecture, correct order, reuses existing abstractions |
| 5 | Optimal architecture, identifies reuse opportunities, perfect ordering |

### 4. Cross-layer coherence (1-5)

Do the data shapes flow correctly between layers (DB → API → Store → UI)?

| Score | Criteria |
|-------|----------|
| 1 | Layers are planned in isolation, shapes will conflict |
| 2 | Some awareness of data flow, gaps between layers |
| 3 | Data flow is implied but not explicit, likely works |
| 4 | Data flow is clear, shapes are compatible across layers |
| 5 | Data contracts are explicit, every layer's input matches the previous layer's output |

### 5. Signature quality (hybrid variant only) (1-5)

Are the typed signatures correct, useful, and well-scoped?

| Score | Criteria |
|-------|----------|
| 1 | Signatures are wrong, use `any` everywhere, or are pseudo-code |
| 2 | Signatures compile but are too loose (broad unions, missing constraints) |
| 3 | Signatures are correct and reasonably typed |
| 4 | Signatures compose well across tasks, use project conventions |
| 5 | Signatures are tight contracts that eliminate implementation ambiguity while remaining flexible |

### 6. Implementability (1-5)

Could a DIFFERENT agent (no prior context) execute this plan and produce working code?

| Score | Criteria |
|-------|----------|
| 1 | Plan requires significant interpretation, many implicit assumptions |
| 2 | Agent would need to ask 3+ clarifying questions |
| 3 | Agent could implement with 1-2 minor assumptions |
| 4 | Agent could implement cold with high confidence |
| 5 | Plan is essentially a deterministic recipe — execution is mechanical |

## Composite Score

**Standard variant:** Average of dimensions 1-4, 6 (5 dimensions, max 25)
**Hybrid variant:** Average of all 6 dimensions (max 30, normalized to 25 for comparison)

## Normalized comparison formula

```
standard_score = (completeness + specificity + architecture + coherence + implementability) / 25
hybrid_score   = (completeness + specificity + architecture + coherence + signatures + implementability) / 30

# Both produce a 0.0-1.0 score for direct comparison
```
