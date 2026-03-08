# spec-plan-ship

**Spec it. Plan it. Ship it.**

A lightweight, battle-tested skill kit for [Claude Code](https://www.anthropic.com/claude-code) that brings structured feature development to your workflow — without the overhead of enterprise frameworks.

Born from combining the best ideas of [BMAD Method](https://docs.bmad-method.org), [Spec-Kit](https://github.com/github/spec-kit), and [Superpowers](https://github.com/obra/superpowers) into something practical for solo developers and small teams.

## What's Inside

| File | Type | What It Does |
|------|------|-------------|
| `skills/feature-spec/SKILL.md` | Skill | **PM mode** — Defines WHAT to build and WHY. Socratic questioning, acceptance criteria, impact analysis. |
| `skills/feature-plan/SKILL.md` | Skill | **Architect mode** — Defines HOW, WHERE, and in what ORDER. Granular tasks with adversarial review. |
| `rules/feature-workflow.md` | Rule | **Orchestrator** — Reminds Claude to follow the spec → plan → ship flow automatically. |

## The Flow

```
1. /feature-spec          →  Spec (WHAT & WHY)
                               ↓
2. /feature-plan [spec]   →  Plan (HOW & WHERE)
                               ↓
                           Adversarial Review (subagent)
                               ↓
3. Implement              →  Execute tasks in order
                               ↓
4. Test → Review → Commit →  Ship it
```

## Why This Exists

Most AI coding workflows skip straight to implementation. That works for small fixes, but for anything non-trivial, you end up:

- Building the wrong thing (no spec)
- Building it wrong (no plan)
- Missing edge cases (no review)

**spec-plan-ship** adds just enough structure to prevent these problems, without slowing you down. Three files. No CLI tools. No dependencies.

## Installation

### Quick Install (Recommended)

Copy the files into your project's `.claude/` directory:

```bash
# From your project root
mkdir -p .claude/skills/feature-spec .claude/skills/feature-plan .claude/rules docs/specs

# Copy skills
curl -sL https://raw.githubusercontent.com/diegosolorzano/spec-plan-ship/main/skills/feature-spec/SKILL.md \
  -o .claude/skills/feature-spec/SKILL.md

curl -sL https://raw.githubusercontent.com/diegosolorzano/spec-plan-ship/main/skills/feature-plan/SKILL.md \
  -o .claude/skills/feature-plan/SKILL.md

# Copy rule
curl -sL https://raw.githubusercontent.com/diegosolorzano/spec-plan-ship/main/rules/feature-workflow.md \
  -o .claude/rules/feature-workflow.md
```

### Manual Install

1. Clone this repo
2. Copy `skills/` and `rules/` into your project's `.claude/` directory
3. Create `docs/specs/` for spec artifacts

```bash
git clone https://github.com/diegosolorzano/spec-plan-ship.git
cp -r spec-plan-ship/skills/ your-project/.claude/skills/
cp -r spec-plan-ship/rules/ your-project/.claude/rules/
mkdir -p your-project/docs/specs
```

### Verify

Open Claude Code in your project. You should see `/feature-spec` and `/feature-plan` in your available skills.

## Usage

### Step 1: Spec (Define WHAT)

```
/feature-spec
```

Claude enters **PM mode** and will:
- Ask clarifying questions (Socratic method, max 3 rounds)
- Scan your codebase for impacted systems
- Generate a spec with requirements, acceptance criteria, and impact analysis
- Save to `docs/specs/{feature-slug}.md`

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
- Save to `.claude/plans/{feature-slug}-plan.md`

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

### Step 3: Ship

Execute the tasks from the plan in order. The workflow rule will remind Claude to follow the process.

## What Each Piece Does

### `/feature-spec` — The PM

Inspired by **Spec-Kit**'s "specifications as executable artifacts" philosophy and **BMAD**'s Product Manager agent.

- Asks questions before assuming (Socratic method)
- Separates WHAT from HOW (specs never include technical decisions)
- Produces a lightweight spec (~1 page) with:
  - Problem statement
  - Functional requirements
  - Acceptance criteria (Given/When/Then)
  - Impacted systems (with file paths from your codebase)
  - Out of scope
  - Open questions

### `/feature-plan` — The Architect

Inspired by **Superpowers**' granular task planning and **BMAD**'s adversarial review process.

- Reads the spec and explores your codebase
- Produces tasks that are:
  - **2-5 minutes** of focused work each
  - **Atomic** — one clear thing to do
  - **Ordered** by dependencies (database → API → UI)
  - **Verifiable** — includes how to confirm it works
- **Adversarial review**: Before presenting the plan, launches a subagent that critiques it for completeness, architecture, ordering, risks, and scope. Incorporates findings automatically.

### `feature-workflow.md` — The Orchestrator

A lightweight rule that reminds Claude to follow the flow when implementing non-trivial changes. Skips automatically for config tweaks, typo fixes, and one-line changes.

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

The workflow rule references optional skills for testing and review. Add your own or use community skills:

```markdown
# In rules/feature-workflow.md
4. **Test** — Use `/tdd-workflow` for tasks requiring tests
5. **Review** — Use `/code-review` for quality check
6. **Commit** — Use `/semantic-commit`
```

### Changing the Model

Both skills default to `model: opus` for maximum reasoning quality. You can change this:

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

### Why Lightweight Specs?

Spec-Kit's full workflow includes constitution, specification, plan, tasks, and implementation phases with a CLI tool. We found that for solo devs, a ~1 page spec with clear acceptance criteria is enough. The overhead of managing multiple artifact types isn't worth it when you're the only stakeholder.

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
