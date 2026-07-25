# Agent Docs Standard (Medium+ features)

Normative templates and criteria for the Agent Readiness deliverables of a Medium+ feature.
Repo-agnostic: nothing here assumes a specific project. Read this when drafting the
`## Agent Readiness` section of a spec, and when materializing its deliverables in a plan.

Sources: https://agents.md/ · https://agentskills.io/specification

## Applicability matrix

Deliverables are mandatory **by default**, but each has a deterministic applicability test.
Every "N/A" requires an explicit 1-line justification in the spec — never a silent omission.

| Deliverable | Required when | N/A only when |
|---|---|---|
| **Skill** | Always for a Medium+ feature | The feature is trivial, or an existing skill/rule fully covers it — **name which one** |
| **AGENTS.md** (+ CLAUDE.md symlink) | The feature **creates or modifies a module/facade folder** | No module folder is touched (e.g. the change lives inside existing docs/skills) |
| **Rule** | The feature introduces a **new always-on constraint** agents must respect on every turn | No new always-on constraint (area-specific guidance belongs in the skill instead) |

A justified N/A is a first-class answer: state it in the spec and the plan quotes it.
An unjustified N/A is a gap the plan must flag, not accept.

## Edit-before-create default

Default to **updating the existing artifact**; create a new one ONLY for a genuinely new
subsystem, folder, or constraint:

- **Skill:** extend the owning subsystem skill when the feature is a facet of it; create a
  new skill only for a distinct subsystem with its own trigger files.
- **AGENTS.md:** update the folder's existing AGENTS.md; create one only for a genuinely
  new module/facade folder.
- **Rule:** update the existing rule that owns the constraint; create a new rule only for a
  genuinely new always-on constraint.

Proliferating near-duplicate skills/rules/AGENTS.md files fragments guidance and poisons
discovery — the anti-pattern this default exists to prevent.

## AGENTS.md template

One short `AGENTS.md` per module/facade folder (closest-wins: the agent editing that folder
reads the local guide, not the whole map). 25-60 lines is the expected size — **guidance,
not a hard gate**. Structure:

1. **Purpose** — 1 line: what this folder is.
2. **Public surface** — the facade entrypoint(s); state that external consumers import
   ONLY from here.
3. **Non-obvious local invariants** — constraints the code cannot express.
4. **Gotchas** — concrete traps, with the fix.
5. **Pointer** — the owning skill/rule/spec where the normative detail lives.

**Forbidden:** file inventories of internal files (they rot and poison context), and
duplicating the owning skill's content — link to it instead.

**CLAUDE.md symlink (this workflow's protocol — not part of the public agents.md
standard):** `CLAUDE.md` is ALWAYS a symlink to `AGENTS.md` — one content, two names,
compatible with Claude Code and other agents. Verify with:

```sh
test -f AGENTS.md && [ "$(readlink CLAUDE.md)" = "AGENTS.md" ]
```

## Skill conformance

A **portable subset** of the agentskills.io specification — NOT a claim of full
conformance. Minimum checks, verifiable anywhere:

- `name` in frontmatter == the skill's directory name, kebab-case.
- Valid YAML frontmatter with at least `name` and `description`.
- `description` states WHAT the skill does + WHEN to use it, with trigger keywords;
  max 1024 characters.
- Body under 500 lines. Bulk detail goes to `references/` files loaded via an explicit
  trigger in the body ("read `references/X.md` IF Y") — progressive disclosure.
- When the official validator is available, run `skills-ref validate <skill-dir>` as the
  primary check and resolve its findings. Note: the official spec defines `allowed-tools`
  as space-separated; some existing skills use commas — a known divergence to resolve
  when validating.

## Lightweight-rule criterion

An always-on rule's whole body is paid on every turn. The decision axis is NOT "is this
important?" but **"can this invariant be violated on ANY turn in the repo?"** (security,
git, data safety). Only then does it earn a rule; subsystem knowledge goes to a skill
(loaded on demand); folder-local guidance goes to that folder's AGENTS.md (closest-wins).

Required form: **signposting** — 1-line invariants (the WHAT) + a pointer "full detail →
skill X". The why, incidents, and procedures live in the skill. Test every line: "is this
needed on EVERY turn?" If not, move it to the skill. (Measured anti-pattern: rules
duplicating their twin skill's full normative body wasted ~9k tokens/turn in one audited
repo; a move-never-delete diet recovered 26% of the always-on budget.)

When a spec adds a rule, its Agent Readiness must state: (a) why the constraint passes the
every-turn filter, (b) the signpost version of the rule, (c) which skill receives the
detail.

Platform note: auto-loaded rule directories (e.g. `.claude/rules/` with `paths:` gating)
are NOT native to Claude Code — they rely on per-repo wiring. Don't assume them in a
consuming repo; the native on-demand mechanism is the skill.

## Optional CI gate

If the target repo has a deterministic module manifest (a list of facade/module folders),
wire a CI check that requires, for each manifest dir, `AGENTS.md` + a `CLAUDE.md` symlink
pointing to it. Reference implementation: nodo-ia's
`scripts/checks/check-facade-deep-imports.sh` (`[agents-md]` block) and its test suite.
This gate is a **pattern, not a requirement** — repos without gate infrastructure rely on
the plan's Verify commands instead.
