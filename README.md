<p align="center">
  <img src="docs/banner.svg" alt="spec-plan-ship — Spec it. Plan it. Ship it." width="800">
</p>

A lightweight, battle-tested skill kit for [Claude Code](https://www.anthropic.com/claude-code) that brings structured feature development to your workflow — without the overhead of enterprise frameworks.

Born from combining the best ideas of [BMAD Method](https://docs.bmad-method.org), [Spec-Kit](https://github.com/github/spec-kit), and [Superpowers](https://github.com/obra/superpowers) into something practical for solo developers and small teams.

## What's Inside

| File | Type | What It Does |
|------|------|-------------|
| `skills/feature-spec/SKILL.md` | Skill | **PM + Tech Lead mode** — Defines WHAT, WHY, and key design decisions. Socratic questioning, acceptance criteria, design decisions, risks, deploy checklist. Detail scales to complexity. |
| `skills/feature-plan/SKILL.md` | Skill | **Architect mode** — Defines HOW, WHERE, and in what ORDER. Granular tasks with adversarial review. Includes an embedded Test Matrix for test design. |
| `skills/test-plan/SKILL.md` | Skill | **QA Architect mode** (Medium+ features) — Expands the plan's Test Matrix into a comprehensive standalone test suite with E2E flows, fixtures, factories, and parallelization groups. |
| `skills/operational-test-plan/SKILL.md` | Skill | **Operational QA mode** — Defines realistic business-flow scenarios and ship gates for workflow features. Proves the product works in operation, not just that code paths pass. |
| `skills/tdd/SKILL.md` | Skill | **QA mode** — RED-GREEN-REFACTOR cycle. Consumes test plan or Test Matrix. Tests pure functions even when marked `Tests: No`. |
| `rules/feature-workflow.md` | Rule | **Orchestrator** — Tiered workflow: spec → plan (with Test Matrix) → test-plan → operational-test-plan when needed → implement → validate → ship. Supports parallel task execution. |
| `rules/operational-functional-testing.md` | Rule | **Ship gate** — Requires operational functional tests for Medium+ features that change real product workflows. |
| `rules/documentation-scope.md` | Rule | **Documentation routing** — Clarifies when to update README, docs, AGENTS, rules, skills, issues, or changelogs. |

## The Flow

```
1. /feature-spec          →  Spec (WHAT & WHY)
                               ↓
2. /feature-plan [spec]   →  Plan (HOW & WHERE) + Test Matrix
                               ↓
                           Adversarial Review (subagent)
                               ↓
3. /test-plan [plan]      →  Full test suite (Medium+ only)
       ↓                        Low complexity? Skip — Test Matrix is enough
                            Adversarial Review (subagent)
                                ↓
4. /operational-test-plan →  Operational scenarios for workflow features
       ↓                        Skip when no real product workflow changes
                                ↓
5. /tdd [task]            →  Write tests from test plan or Test Matrix
       ↓                        (RED → GREEN → REFACTOR)
       ↓                        Independent tasks run in parallel
6. Implement              →  Execute tasks, tests guide you
                                ↓
7. Validate ops → Review  →  Commit and ship
```

### Development tiers (Lite / Standard / Full)

Every change is classified by structural blast radius — after exploring what it touches,
re-checked at planning. Full table and modifiers: `skills/feature-spec/references/tier-policy.md`.

| Tier | Protocol | Test approach |
|------|----------|---------------|
| **Full** — entry points, outbound requests, permissions/secrets, money/state machines, schema transforms, architecture contracts | everything, nothing skipped | `/test-plan` |
| **Standard** — feature inside an existing subsystem, additive schema, blocking CI, major dep upgrade | short spec → plan → TDD → E2E | Test Matrix in plan; `/operational-test-plan` directly from the plan when it changes a real workflow |
| **Lite** — copy/styles/docs/config, local fixes, patch/minor deps | implement directly | verification by change type |

In doubt → the higher tier; discovering a higher trigger mid-change promotes the change. The security-review gate and the LLM-evals modifier apply in every tier.

### When to use `/operational-test-plan`

Use it when a feature changes a real operating workflow: business capability, conversational pipeline, admin/operator handoff, webhook, queue, cron, external integration, state machine, or multi-step user journey. Full tier runs it after `/test-plan`; Standard tier runs it directly from the plan.

Technical E2E asks "does the UI/API path run?" Operational testing asks "does the product actually work for the business scenario?" A restaurant order flow, for example, is not shipped because the menu API and order page pass; it is shipped when realistic customer messages create the right pending order, the operator can act on it, and the customer receives the right outcome.

## Why This Exists

Most AI coding workflows skip straight to implementation. That works for small fixes, but for anything non-trivial, you end up:

- Building the wrong thing (no spec)
- Building it wrong (no plan)
- Missing edge cases (no review)

**spec-plan-ship** adds just enough structure to prevent these problems, without slowing you down. Skills and rules only. No CLI tools. No dependencies.

## Installation

### Quick Install (Recommended)

Copy the files into your project's `.claude/` directory:

```bash
# From your project root
mkdir -p .claude/skills/feature-spec .claude/skills/feature-plan .claude/skills/test-plan .claude/skills/operational-test-plan .claude/skills/tdd .claude/rules docs/specs

# Copy skills
curl -fsSL https://raw.githubusercontent.com/diegoesolorzano/spec-plan-ship/main/skills/feature-spec/SKILL.md \
  -o .claude/skills/feature-spec/SKILL.md

mkdir -p .claude/skills/feature-spec/references
curl -fsSL https://raw.githubusercontent.com/diegoesolorzano/spec-plan-ship/main/skills/feature-spec/references/agent-docs-standard.md \
  -o .claude/skills/feature-spec/references/agent-docs-standard.md

curl -fsSL https://raw.githubusercontent.com/diegoesolorzano/spec-plan-ship/main/skills/feature-plan/SKILL.md \
  -o .claude/skills/feature-plan/SKILL.md

curl -fsSL https://raw.githubusercontent.com/diegoesolorzano/spec-plan-ship/main/skills/test-plan/SKILL.md \
  -o .claude/skills/test-plan/SKILL.md

curl -fsSL https://raw.githubusercontent.com/diegoesolorzano/spec-plan-ship/main/skills/operational-test-plan/SKILL.md \
  -o .claude/skills/operational-test-plan/SKILL.md

curl -fsSL https://raw.githubusercontent.com/diegoesolorzano/spec-plan-ship/main/skills/tdd/SKILL.md \
  -o .claude/skills/tdd/SKILL.md

# Copy rule
curl -fsSL https://raw.githubusercontent.com/diegoesolorzano/spec-plan-ship/main/rules/feature-workflow.md \
  -o .claude/rules/feature-workflow.md

curl -fsSL https://raw.githubusercontent.com/diegoesolorzano/spec-plan-ship/main/rules/operational-functional-testing.md \
  -o .claude/rules/operational-functional-testing.md

curl -fsSL https://raw.githubusercontent.com/diegoesolorzano/spec-plan-ship/main/rules/documentation-scope.md \
  -o .claude/rules/documentation-scope.md
```

### Manual Install

1. Clone this repo
2. Copy `skills/` and `rules/` into your project's `.claude/` directory
3. Create `docs/specs/` for spec artifacts

```bash
git clone https://github.com/diegoesolorzano/spec-plan-ship.git
cp -r spec-plan-ship/skills/ your-project/.claude/skills/
cp -r spec-plan-ship/rules/ your-project/.claude/rules/
mkdir -p your-project/docs/specs
```

### Verify

Open Claude Code in your project. You should see `/feature-spec`, `/feature-plan`, `/test-plan`, `/operational-test-plan`, and `/tdd` in your available skills. The `feature-spec` skill also bundles `references/agent-docs-standard.md` (the Agent Readiness standard) — verify it exists at `.claude/skills/feature-spec/references/`.

## Usage

### Step 1: Spec (Define WHAT)

```
/feature-spec
```

Claude enters **PM mode** and will:
- Ask clarifying questions (Socratic method, max 3 rounds)
- Scan your codebase for impacted systems
- Generate a spec with requirements, acceptance criteria, and impact analysis
- Save to `docs/specs/{id}.md`

**Example:**
```
You: /feature-spec
You: I want to add email notifications when an order is completed

Claude (as PM):
- Who receives these? The buyer only, or also admins?
- What triggers "completed"? Payment confirmed or deliverable ready?
- Should it work for guest checkout too?
...produces spec with requirements and acceptance criteria...
```

### Step 2: Plan (Define HOW)

```
/feature-plan docs/specs/email-notifications.md
```

Claude enters **Architect mode** and will:
- Read the approved spec
- Explore your codebase for existing patterns to reuse
- Break implementation into 2-5 minute tasks with exact file paths
- Run an adversarial review via subagent before presenting
- Save to `docs/specs/{id}-plan.md`

**Example output:**
```markdown
### Task 1: Create email templates
- Files: `src/lib/email/templates/order-complete.ts` (create)
- Do: Create HTML email template following pattern in `templates/welcome.ts`
- Verify: Template renders correctly with test data
- Depends on: None

### Task 2: Add notification trigger
- Files: `src/app/api/orders/complete/route.ts` (modify)
- Do: After status update to "completed", call sendOrderComplete()
- Verify: POST to endpoint triggers email (check logs)
- Depends on: Task 1
```

### Step 3: Test Plan (Medium+ Features Only)

For features with cross-layer interactions, multiple consumers, or shared state:

```
/test-plan docs/specs/email-notifications-plan.md
```

Claude enters **QA Architect mode** and will:
- Read the approved plan's Test Matrix and expand it
- Discover existing test infrastructure (framework, patterns, fixtures)
- Design detailed test cases with code-level fixtures and factory functions
- Add E2E flows for cross-layer integration testing
- Map which tests can run in parallel
- Run an adversarial review via subagent before presenting
- Save to `docs/specs/{id}-tests.md`

**For Low complexity features, skip this step** — the Test Matrix embedded in the plan (from Step 2) provides sufficient test design.

**Philosophy: Design tests before code.** When you write tests during implementation, you test what you built. When you design tests upfront, you test what you should have built.

### Step 4: Operational Test Planning

For Medium+ workflow features, add the operational gate before implementation:

```
/operational-test-plan docs/specs/email-notifications-plan.md
```

Claude enters **Operational QA mode** and will:
- Identify the real business capabilities the feature must prove
- Define realistic actor/system scenarios across state, UI, side effects, and handoffs
- Separate automated operational tests, staging smoke, manual acceptance, and future automation
- Save to `docs/specs/{id}-ops.md`

### Step 5: Test-Driven Implementation

For each plan task, `/tdd` consumes the pre-designed test cases:

```
/tdd "user can reset password with valid token"
```

Claude enters **QA mode** and will:
- Check the test plan for pre-designed cases (preferred over ad-hoc)
- Write failing tests FIRST (RED)
- Implement minimum code to pass (GREEN)
- Refactor with green tests as safety net (REFACTOR)
- Cover unit, integration, AND E2E tests against real services

**Parallel execution:** Tasks with no dependency conflicts run concurrently via subagents. Each subagent receives its task + test cases and follows the TDD cycle independently.

**Philosophy: Real over mocked.** Tests run against your actual database, real auth, real file system. Mocks are only for external paid APIs (Stripe, OpenAI, etc.).

### Step 6: Ship

Execute remaining tasks from the plan. For Medium+ workflow features, run the blocking scenarios from `docs/specs/{id}-ops.md` before marking the feature shipped.

## What Each Piece Does

### `/feature-spec` — The PM

Inspired by **Spec-Kit**'s "specifications as executable artifacts" philosophy and **BMAD**'s Product Manager agent.

- Asks questions before assuming (Socratic method)
- Captures WHAT, WHY, and key design decisions
- Detail scales to complexity — simple features get short specs, complex features get comprehensive specs
- Produces a spec with:
  - Problem statement and background context
  - Functional requirements and acceptance criteria (Given/When/Then)
  - Scope grouped by area with file paths
  - Design decisions with reasoning
  - Schema, API, and UI changes
  - Risks with mitigation and rollback
  - Deploy checklist

### `/feature-plan` — The Architect

Inspired by **Superpowers**' granular task planning and **BMAD**'s adversarial review process.

- Reads the spec and explores your codebase
- Produces tasks that are:
  - **2-5 minutes** of focused work each
  - **Atomic** — one clear thing to do
  - **Ordered** by dependencies (database → API → UI)
  - **Verifiable** — includes how to confirm it works
- **Test Matrix** (full mode): Maps acceptance criteria to test cases, defines shared fixtures, includes contract-under-test signatures, and adds E2E flows for cross-layer features
- **Adversarial review**: Before presenting the plan, launches a subagent that critiques it for completeness, architecture, ordering, test coverage, risks, and scope. Incorporates findings automatically.

### `/test-plan` — The QA Architect (Medium+ only)

For features with cross-layer interactions, multiple consumers, or shared state. Expands the plan's Test Matrix into a standalone document.

- Reads the approved plan and corresponding spec
- Discovers existing test infrastructure (framework, patterns, fixtures)
- Expands Test Matrix cases into code-level fixtures, factory functions, and custom matchers
- Designs E2E flows testing cross-task integration boundaries
- Maps parallelization groups (which test files can run independently)
- **Adversarial review**: Before presenting, a subagent critiques coverage gaps, test quality, and infrastructure assumptions
- Produces a test plan that `/tdd` consumes during implementation

Not needed for Low complexity features — the plan's embedded Test Matrix is sufficient.

### `/operational-test-plan` — The Operational QA Architect

For Medium+ features that change real product workflows. It proves the business capability works across actors, state, side effects, and handoffs.

- Reads the spec, plan, and test plan when present
- Identifies operational capabilities and stable test boundaries
- Designs realistic scenarios across users, operators/admins, automation, and integrations
- Separates automated operational tests, staging smoke, manual acceptance, and future automation
- Produces `docs/specs/{id}-ops.md` with blocking ship gate evidence

Not needed for isolated code changes with no operational behavior.

### `/tdd` — The QA Engineer

Inspired by **Superpowers**' TDD enforcement and the principle that real tests beat mocked tests.

- **Consumes test artifacts** — checks for a full test plan first, then the plan's Test Matrix, then designs ad-hoc
- **Pure function override** — tests pure functions even when the plan marks a task `Tests: No`
- Follows RED → GREEN → REFACTOR strictly
- Tests at all levels: unit, integration, E2E
- **E2E tests are mandatory for user-facing features** — not optional
- Tests against real services by default (database, auth, file system)
- Only mocks external paid APIs (Stripe, OpenAI, etc.)

### `feature-workflow.md` — The Orchestrator

A tiered rule that reminds Claude to follow the flow when implementing non-trivial changes. Test depth scales with feature complexity: Test Matrix for all features, full test plan for Medium+, operational test plan for workflow features. Skips automatically for config tweaks, typo fixes, and one-line changes. Supports parallel execution of independent tasks via subagents.

## Customization

### Adapting to Your Project

The skills reference generic paths. Update them to match your project structure:

**In `feature-spec/SKILL.md`**, update Step 2 (Explore Impacted Systems) with your actual directories:
```markdown
- Database: `database/migrations/`        → your migration path
- API routes: `src/app/api/`              → your API path
- Components: `src/components/`           → your component path
- State stores: `src/stores/`             → your store path
```

**In `feature-plan/SKILL.md`**, update Step 2 (Deep Codebase Exploration) with your domain rules and patterns.

### Extending the Flow

The workflow includes spec, plan, test planning, operational validation, TDD, and implementation. You can add more skills for review and commit:

```markdown
# In rules/feature-workflow.md — add near the end:
8. **Review** — Use `/code-review` for quality check
9. **Commit** — Use `/semantic-commit` for conventional commits
```

### Changing the Model

Planning skills default to `model: opus` for maximum reasoning quality. You can change this:

```yaml
# In SKILL.md frontmatter
model: sonnet  # Faster, good for straightforward features
model: opus    # Best reasoning, good for complex features
```

## Design Decisions

### Why Skills, Not Agents?

BMAD uses separate agents (PM, Architect, Dev, QA). We use **skills with role prompts** instead because:
- Claude already has the knowledge of all these roles
- Skills run in the main conversation (shared context)
- No agent orchestration overhead
- You control when to switch "hats"

### Why Adversarial Review as Subagent?

The adversarial review in `/feature-plan` uses a Claude Code subagent (not a separate tool) because:
- It has full access to your codebase
- It shares the same model capabilities
- No external dependencies
- Can be extended later to also use a different model for diversity of opinion

### Why Scalable Specs?

Spec-Kit's full workflow includes constitution, specification, plan, tasks, and implementation phases with a CLI tool. We found that for solo devs, a spec that scales to the feature's complexity works best — simple changes get brief specs, complex features get comprehensive specs with full decision context. The overhead of managing multiple artifact types isn't worth it when you're the only stakeholder, but under-documenting complex decisions leads to "why did we do this?" questions later.

## Community Integrations

spec-plan-ship is tool-agnostic by design — it works with Claude Code alone. But if you use other AI coding tools, you can add external review agents for diversity of opinion:

### OpenCode (cross-model review)

If you have [OpenCode](https://opencode.ai) installed, you can send plans, code, and test artifacts to a different model for independent review:

| Agent | What It Reviews | Example |
|-------|----------------|---------|
| `plan-reviewer` | Implementation plans | `opencode run --agent plan-reviewer --dir . "Review this plan" -f docs/specs/my-plan.md` |
| `readonly-code-reviewer` | Code changes | `opencode run --agent readonly-code-reviewer --dir . "Review recent changes"` |
| `test-reviewer` | Test plans and test suites | `opencode run --agent test-reviewer --dir . "Review test coverage" -f docs/specs/my-tests.md` |

These agents operate in read-only mode — they analyze and report, never modify files. The value is getting a second opinion from a different model (GPT, Gemini, etc.) before implementing.

To set up, create agent files in `~/.config/opencode/agents/agent/` — see [OpenCode docs](https://opencode.ai) for agent configuration.

## Credits & Inspiration

This kit wouldn't exist without these projects:

| Project | What We Took | Link |
|---------|-------------|------|
| **BMAD Method** | Role-based thinking (PM/Architect), adversarial review, progressive context building | [docs.bmad-method.org](https://docs.bmad-method.org) |
| **Spec-Kit** | Spec-driven development, "unit tests for English" (acceptance criteria as checklists), WHAT/HOW separation | [github.com/github/spec-kit](https://github.com/github/spec-kit) |
| **Superpowers** | 2-5 minute granular tasks, plan-first workflow, skill composability | [github.com/obra/superpowers](https://github.com/obra/superpowers) |

## Requirements

- [Claude Code](https://www.anthropic.com/claude-code) (CLI)
- A project with a `.claude/` directory

No other dependencies. No CLI tools. No build steps.

## License

MIT
