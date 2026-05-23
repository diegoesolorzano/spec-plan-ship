# Eval Case 01: REST API Endpoint

**Complexity:** Low-Medium
**Stack:** Node.js + Express + PostgreSQL
**Why this case:** Tests basic CRUD planning — the bread and butter of feature plans.

## Simulated Project Context

```
src/
  routes/
    users.ts          # GET /users, POST /users (existing)
    products.ts       # GET /products (existing)
  lib/
    db.ts             # Knex instance export
    auth.ts           # requireAuth middleware
  types/
    user.ts           # User interface
    product.ts        # Product interface
```

## Spec (Input)

Feature: Add a "favorites" system where users can save products to a favorites list.

### Requirements
- FR-1: User can add a product to favorites (POST /api/favorites)
- FR-2: User can remove a product from favorites (DELETE /api/favorites/:id)
- FR-3: User can list their favorites (GET /api/favorites)
- FR-4: Duplicate favorites are rejected (409 Conflict)
- FR-5: Only authenticated users can manage favorites

### Acceptance Criteria
- Given an authenticated user, When they POST a valid productId, Then the favorite is created and 201 returned
- Given an authenticated user, When they POST a duplicate productId, Then 409 is returned
- Given an authenticated user, When they GET /favorites, Then only their favorites are returned (not other users')
- Given an unauthenticated request, When they hit any favorites endpoint, Then 401 is returned

## Scoring Dimensions

Rate 1-5 for each:

| Dimension | What to evaluate |
|-----------|-----------------|
| **Completeness** | Are all 5 FRs covered? Is the migration included? Auth middleware referenced? |
| **Specificity** | Can an agent implement each task without asking clarifying questions? |
| **Architecture** | Does the plan follow existing patterns (routes/, lib/, types/)? Correct dependency order? |
| **Signature quality** (hybrid only) | Are types correct? Do they compose? Are they over/under-specified? |
| **Implementability** | Could a different agent execute this plan cold and produce working code? |
