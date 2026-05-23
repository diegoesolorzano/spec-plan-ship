# Eval Case 03: Extract and Refactor Shared Module

**Complexity:** Medium
**Stack:** TypeScript monorepo (Turborepo) — shared package extraction
**Why this case:** Tests planning for code that already exists and needs restructuring. No new features — pure architectural change. Exposes whether the plan can describe "move + reshape" operations clearly.

## Simulated Project Context

```
packages/
  web/
    src/
      lib/
        pricing.ts        # calculatePrice(), applyDiscount(), formatCurrency()
        pricing.test.ts   # Tests for above (existing)
      components/
        PriceDisplay.tsx   # Uses pricing.ts directly
        CartSummary.tsx    # Uses pricing.ts directly
  mobile/
    src/
      utils/
        pricing.ts        # DUPLICATED: calculatePrice(), applyDiscount() (slightly different)
        format.ts         # formatCurrency() (different implementation)
  shared/
    src/
      index.ts            # Exports from shared (currently: types only)
    package.json          # @repo/shared
    tsconfig.json
turbo.json
package.json              # Root workspace config
```

## Spec (Input)

Feature: Extract pricing logic into `@repo/shared` to eliminate duplication between web and mobile.

### Requirements
- FR-1: Create `packages/shared/src/pricing.ts` with canonical implementations
- FR-2: Reconcile the two `calculatePrice()` variants — mobile has a rounding bug, web is correct
- FR-3: Reconcile `formatCurrency()` — mobile supports more locales, web version is simpler. Keep mobile's broader support.
- FR-4: Update web imports to use `@repo/shared`
- FR-5: Update mobile imports to use `@repo/shared`
- FR-6: Delete duplicated files from web and mobile
- FR-7: Existing tests must pass (migrate web's tests to shared, verify mobile's edge cases are covered)

### Acceptance Criteria
- Given the shared package is built, When web imports calculatePrice, Then it gets the correct (web) implementation
- Given mobile previously had a rounding bug, When mobile imports from shared, Then the bug is fixed
- Given mobile's formatCurrency supported 12 locales, When shared's formatCurrency is used, Then all 12 still work
- Given web's pricing.test.ts, When moved to shared, Then all tests pass without modification (except import paths)
- Given the refactor is complete, When searching for calculatePrice across the monorepo, Then it exists only in @repo/shared

## Scoring Dimensions

Rate 1-5 for each:

| Dimension | What to evaluate |
|-----------|-----------------|
| **Completeness** | All 7 FRs covered? Test migration included? Dead code cleanup included? |
| **Specificity** | Is it clear WHICH implementation wins for each function? Are the reconciliation rules explicit? |
| **Architecture** | Does the plan address Turborepo dependency graph? tsconfig paths? Package exports? |
| **Risk awareness** | Breaking changes across packages identified? Build order dependencies? |
| **Signature quality** (hybrid only) | Does the shared module's export surface match what both consumers need? Are generic signatures used where web and mobile differ? |
| **Implementability** | Could a different agent execute this without knowing which variant is "correct"? |
