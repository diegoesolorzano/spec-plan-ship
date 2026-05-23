# Eval Runner

How to run plan variant evaluations using Claude Code.

## Prerequisites

- Both skills installed (or accessible from this repo):
  - `skills/feature-plan/SKILL.md` (standard — Approach B)
  - `skills/feature-plan-hybrid/SKILL.md` (hybrid — Approach C)
- Eval cases in `evals/cases/`
- Scoring rubric in `evals/scoring-rubric.md`

## Running an Eval

### Step 1: Generate plans (one per variant per case)

For each case file in `evals/cases/`, run both variants:

```
# Standard variant
Read evals/cases/case-01-api-endpoint.md, then act as the feature-plan skill
(load skills/feature-plan/SKILL.md instructions). Generate a plan for the spec
described in the case. Use the simulated project context as your codebase knowledge.
Save the plan to evals/results/case-01-standard.md

# Hybrid variant
Read evals/cases/case-01-api-endpoint.md, then act as the feature-plan-hybrid skill
(load skills/feature-plan-hybrid/SKILL.md instructions). Generate a plan for the spec
described in the case. Use the simulated project context as your codebase knowledge.
Save the plan to evals/results/case-01-hybrid.md
```

Important: skip the adversarial review subagent during evals — we're measuring the plan generation quality, not the review loop.

### Step 2: Score plans

For each generated plan, run the scorer prompt:

```
Read evals/scoring-rubric.md for the scoring criteria.
Read evals/results/case-01-standard.md (the plan to score).
Read evals/cases/case-01-api-endpoint.md (the original spec for context).

Score this plan on each dimension from the rubric. For each dimension:
1. State the score (1-5)
2. Give one sentence of justification
3. Quote the specific part of the plan that supports your score

Output as a markdown table. Append to evals/results/case-01-standard.md under
a "## Eval Score" heading.
```

### Step 3: Compare

After scoring all plans, generate a comparison table:

```
Read all scored plans in evals/results/.
Generate a comparison summary at evals/results/COMPARISON.md with:
- Side-by-side scores per case
- Normalized composite scores
- Which variant won per dimension
- Overall recommendation with reasoning
```

## Batch Run (Recommended)

Use parallel subagents to generate all plans simultaneously:

```
Launch 6 agents (3 cases x 2 variants):
- Agent 1: case-01 + standard → evals/results/case-01-standard.md
- Agent 2: case-01 + hybrid  → evals/results/case-01-hybrid.md
- Agent 3: case-02 + standard → evals/results/case-02-standard.md
- Agent 4: case-02 + hybrid  → evals/results/case-02-hybrid.md
- Agent 5: case-03 + standard → evals/results/case-03-standard.md
- Agent 6: case-03 + hybrid  → evals/results/case-03-hybrid.md

Then score all 6 sequentially (scorer needs consistency, parallelism hurts).
Then generate COMPARISON.md.
```

## Interpreting Results

**Hybrid wins if:** signature quality adds enough implementability and coherence improvement to justify the heavier plan phase.

**Standard wins if:** the extra specificity from signatures doesn't materially improve implementation outcomes — meaning prose instructions were already sufficient.

**Key signal:** The gap in dimension 6 (implementability) matters most. If hybrid scores significantly higher on implementability, the signatures are doing real work. If scores are similar, the signatures are ceremony.
