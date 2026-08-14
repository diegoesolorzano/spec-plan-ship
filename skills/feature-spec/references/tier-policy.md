# Development Tier Policy (Lite / Standard / Full)

Normative reference for classifying every change by **structural blast radius** and paying
only the protocol its risk justifies. Self-contained: everything needed to classify lives in
this file. Consumed by `feature-spec` (classification step), `feature-plan`
(reclassification), `test-plan` (tier guard), and the `feature-workflow` rule (summary
table points here).

## When to classify

Classification happens **after** understanding the request and exploring the impacted
systems — the criteria require knowing what the change touches — and **before** drafting any
artifact. It is **re-evaluated when planning**, once concrete files and contracts are known.
Never classify from the diff size; classify from what the change *touches*.

## Decision table (precedence-ordered)

Evaluate top to bottom. The first matching row wins. Transversal modifiers (below) are
evaluated **always, in every tier**.

### 1. Full — any of these triggers → Full, no exceptions

| # | Trigger |
|---|---------|
| F1 | New or modified entry point: endpoint, webhook, cron, queue consumer, RPC, CLI command, agent-callable tool |
| F2 | Outbound request to a service you do not control (new or changed external integration) |
| F3 | Permissions, roles, access policies (RLS or equivalent), secrets, tokens, audiences |
| F4 | Money, or a business state machine |
| F5 | Schema migration with data transformation, or a data backfill |
| F6 | Architecture: new facade, new or changed canonical capability, cross-service contract, shared-library change that alters the contract of 2+ consumers |

**Full protocol:** spec → plan → test-plan → operational-test-plan (when the feature changes
a real workflow) → TDD → E2E → security review. Nothing is skipped.

### 2. Standard — no Full trigger, and at least one of

| # | Trigger |
|---|---------|
| S1 | Feature or extension over existing patterns (new branch of existing logic, new page over existing APIs, extending a function without changing permissions) |
| S2 | **Additive** schema migration with no data transformation (new table/column with the repo's standard isolation policies) |
| S3 | Shared-library change that does NOT alter any consumer contract |
| S4 | CI/infrastructure change on a blocking path (gates, deploys) |
| S5 | **Major** dependency upgrade |
| S6 | Multi-file bug fix crossing modules |

**Standard protocol:** short spec (required reduced section set: Problem Statement,
Requirements, Acceptance Criteria, Sprint Goal; body ≤ 60 lines) → plan → TDD → E2E. No
standalone test-plan (the plan's embedded Test Matrix suffices). A Standard feature that
changes a real product workflow invokes `operational-test-plan` **directly from the plan**.

### 3. Lite — no Full or Standard trigger

Copy/styles/UI with no new logic, docs, declarative config, a bug fix local to one module,
patch/minor dependency upgrades.

**Eligibility condition (non-circular):** the adopting repo **declares the list of
deterministic gates** that constitute Lite's safety net (typecheck, unit suites, invariant
linters…) in its adoption note. Evidence of coverage = that list running green in CI for the
change.

**Lite protocol:** implement directly. Verification by change type:

| Lite change type | Required verification |
|---|---|
| Logic (local fix, config with runtime effect) | Unit test covering the change |
| UI/styles | Visual verification per the repo's visual workflow |
| Copy/docs | None beyond deterministic gates + review |
| No testable surface | Declare "no test surface" + name the deterministic gate that covers it |

**Security caveat (always):** permissions and secrets are a **Full trigger**, and the
security-review gate applies in EVERY tier — a one-line `GRANT` is a Lite-looking change and
is exactly the class of bug that gate exists for. Tiers reduce artifacts; they never disable
a security gate.

## Boundary cases (pre-labeled)

| Change | Tier | Why |
|---|---|---|
| Additive migration, no data transform | Standard | S2 |
| Migration with backfill | Full | F5 beats S2 (precedence) |
| Minor/patch dependency upgrade | Lite | no trigger |
| Major dependency upgrade | Standard | S5 |
| One-line system-prompt edit | Lite **+ llm-evals** | no structural trigger; LLM modifier |
| Blocking CI change | Standard | S4 |
| Multi-file bug fix | Standard | S6 |
| Shared lib, no contract change | Standard | S3 |
| Shared lib, contract change | Full | F6 |
| Copy/docs | Lite | no trigger |
| New endpoint + prompt edit | Full + llm-evals | F1; modifier is orthogonal |
| Secret discovered mid-change | Full | promotion rule |
| New data read/write inside a subsystem | Standard + security-review | S1; security modifier |

## Transversal modifiers (any tier, always evaluated)

### LLM modifier (`llm-evals`)

If the change alters the behavior of a **non-deterministic** component — system prompt,
model, inference parameters, retrieval/context, tool wiring, agent logic — it carries
**evals that measure the behavior change**: baseline before, comparison after, zero
regressions on existing cases. A one-line prompt diff is structurally Lite but its blast
radius is the system's entire production behavior, and no deterministic gate measures it.

**Fallback without an eval harness (precise and measurable):** a **protocolized manual A/B**
— a fixed set of ≥10 representative cases of the touched behavior, before/after outputs
recorded and archived with the change, and explicit human sign-off; additionally the missing
harness is registered as debt with an issue. The manual A/B satisfies the modifier.
**Promoting the tier does NOT satisfy it** — tier and behavioral evidence are orthogonal.

### Security modifier (`security-review`)

The security-review gate keeps its own triggers and applies **regardless of tier**: entry
points, outbound requests, permissions/secrets, data reads/writes, schema/data operations. A
Standard change adding a new data access carries its security review even though it does not
pay the rest of the Full protocol.

## Promotion rule (anti-gaming)

- In doubt between two tiers → the higher one.
- If mid-change a higher-tier trigger is discovered, the change **is promoted** — it is
  never finished "because it is almost done". Reclassification at planning time is
  mandatory.

## Recording the classification

Tier metadata goes in the artifact **after the contract header block and one blank line**,
outside the normative six-field block:

```markdown
**Feature ID:** … (six contract fields) …

**Tier:** standard
**Modifiers:** llm-evals
```

## Release-review contract (code review fires per release, not per feature)

The external code review (`code-diff`) fires **once per release candidate** instead of per
feature. Provider-neutral contract:

1. **Reviewed object:** the diff between two immutable SHAs — `base` (current production
   SHA) and `head` (the release candidate SHA). Never moving refs.
2. **Durable identity:** each release has a repo-defined monotonic id. The review lock
   namespace uses `release-{id}` as feature-id with **deterministic round allocation per
   candidate** (round N = existing markers for `release-{id}` + 1, invoked
   `--round N --rounds N`): the marker prevents double-billing; a new candidate consumes the
   next round and never collides.
3. **Invocation owner:** the repo's release ceremony.
4. **Blocking point with durable evidence:** promotion to production requires a completed
   review of the exact promoted `head`, proven by **durable evidence** written by the
   ceremony (base SHA, head SHA, diff digest, round, result) and verified before promotion —
   the lock marker only prevents double-billing; it does NOT prove which SHA was reviewed.
   A fix changes the diff → new `head` → new round. No SHA is promoted without evidence.
5. **Failure/timeout = fail loud:** an empty, timed-out, or errored review **did not
   happen** (exit 3 semantics) and blocks promotion — never degraded to a pass.
6. **Large diffs:** above the repo-declared threshold (suggested default: 8,000 changed
   lines), the ceremony either chunks the review by top-level directory in sequential rounds
   or rejects it demanding a smaller release — never silently reviews a sample.
7. **Incorporation:** stays `propose-patches` — the release review never rewrites code; its
   findings become fixes that produce a new candidate (point 4).
8. **Evidence location:** release-review reports/evidence are NOT archived under
   `docs/specs/{id}-r{round}.md` (that contract requires a Feature ID with an upstream) —
   they live in a repo-defined release location (e.g. `docs/releases/`), outside the feature
   artifact contract.

Portable invocation example (two-step; no shell process substitution):

```
git diff <prodSHA>..<headSHA> > /tmp/release.diff
cmrl review --type code-diff --diff /tmp/release.diff --feature-id release-{id} \
  --round N --rounds N --dir <repo>
```

## Adoption per repo

A repo adopting this policy declares, in its own docs:

1. The **deterministic gate list** that backs Lite.
2. Its **monotonic release id** scheme and release-evidence location.
3. Its **diff threshold** for the release review (or the 8,000-line default).
4. Whether it has an **eval harness** (and where), or that the manual A/B fallback applies.

Policy, its internal classifier, and its fixtures are three artifacts of ONE policy — they
change together, never separately.
