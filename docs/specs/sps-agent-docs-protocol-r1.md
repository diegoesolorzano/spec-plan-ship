# Review: sps-agent-docs-protocol — round 1

**Feature ID:** sps-agent-docs-protocol
**Repo:** spec-plan-ship
**Issue:** none
**Upstream:** docs/specs/sps-agent-docs-protocol.md
**Date:** 2026-07-25
**Status:** InReview
**Review:** real

## Summary
The spec standardizes Agent Readiness deliverables for Medium+ features across `/feature-spec` and `/feature-plan`: conformant skills, local `AGENTS.md` plus `CLAUDE.md` symlinks, lightweight rules, edit-before-create defaults, concrete verification, and synchronized copies across supported surfaces.

## Findings
- **[CRITICAL] FR-4 / public installation path** — The recommended README installation downloads only `SKILL.md`; it will not install the new `references/agent-docs-standard.md`. The resulting public installation would reference a missing file, contradicting the claim that README changes are N/A. Add the reference to Quick Install and verification instructions, and gate installation completeness with a test.
- **[CRITICAL] FR-7 / drift acceptance criterion** — `check-skills-drift.sh` reports drift and missing files only informationally and still exits 0; therefore "script exits 0" does not prove byte identity or even that every surface exists. Change the checker to fail on drift/missing required surfaces, or add explicit `diff` and existence checks plus negative behavior tests.
- **[WARNING] Agent Readiness applicability is contradictory** — FR-1 calls all three artifacts mandatory, while FR-5 says "each applicable," the current feature marks `AGENTS.md` N/A, and rules are inherently conditional. Define a deterministic applicability matrix and require explicit N/A justification; acceptance criteria should verify either the artifact or that justification.
- **[WARNING] Agent Skills conformance is underspecified and under-verified** — The official specification includes frontmatter syntax, name grammar, directory matching, description limits, and other constraints, while the proposed checks primarily enforce `<500` lines. Define the complete required subset and preferably use `skills-ref validate`, supplemented by checks for recommendations the validator does not enforce.
- **[WARNING] AGENTS.md line-count semantics conflict** — FR-1 presents 25–60 lines as mandatory, while Risk Mitigation calls it an expected range rather than a strict schema. Decide whether this is a gate or guidance. If gated, specify the exact command and treatment of blank lines; if guidance, remove it from mandatory conformance language.
- **[WARNING] Symlink verification is insufficient** — `test -L {dir}/CLAUDE.md` accepts broken, absolute, or incorrectly targeted symlinks. Require verification that it resolves specifically to the sibling `AGENTS.md`, remains valid, and contains no duplicate regular file.
- **[WARNING] The global rule update is not consistently scoped** — Agent Readiness requires modifying `~/.claude/rules/agent-docs-on-features.md`, but it is absent from Requirements, the main Scope list, Done-when evidence, and `surfaces.list`. A home-directory-only source also conflicts with the suite's portable, versioned-source convention. Either remove this implementation deliverable or identify its canonical repository source, replicas, and drift checks.
- **[WARNING] Static behavior coverage is missing** — Existing `tests/run.sh` does not validate the new Medium+ trigger, generated Agent Readiness shape, edit-before-create wording across all three artifact types, or concrete feature-plan verification. Add focused behavior/lint tests rather than relying on broad suite success and manual inspection.
- **[WARNING] Several Done-when checks can produce false positives** — Grepping for `CLAUDE.md`, `symlink`, `500`, `test -L`, or `extend|update.*existing` does not prove the required sections, applicability logic, or all three deliverables exist. Replace these with section-aware lint assertions or behavior fixtures containing positive and negative cases.
- **[WARNING] Deployment verification is vague and potentially mutating** — "Run `/feature-spec` in any repo" does not define a fixture, expected output, or cleanup and may create an artifact in an unrelated repository. Use an isolated fixture repository and specify exact assertions.
- **[INFO]** The spec correctly chooses progressive disclosure, edit-before-create, an optional consumer CI gate, explicit out-of-scope boundaries, and cross-surface synchronization as core design principles.

## Verdict
REQUEST_CHANGES — The recommended public installation would omit the required reference, and the stated drift gate cannot currently prove the byte-identical synchronization required by the feature.
