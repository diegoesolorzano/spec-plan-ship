# Review: sps-agent-docs-protocol — round 2 (plan)

**Feature ID:** sps-agent-docs-protocol
**Repo:** spec-plan-ship
**Issue:** none
**Upstream:** docs/specs/sps-agent-docs-protocol-plan.md
**Date:** 2026-07-25
**Status:** InReview
**Review:** real

## Verdict
MAJOR_REWORK

## Improvements
- **[Priority: HIGH]** Fix the already-invalid suite artifact before claiming a green baseline. `spec-plan-ship-suite/docs/specs/handoff-agent-docs-protocol.md` violates the suite key convention: its filename derives `handoff-agent-docs-protocol`, while `AGENTS.md:16-19` requires a `suite-` ID. `scripts/check-artifact-contract.sh .` currently fails, contradicting Task 1's claim that `tests/run.sh` remains green. Rename/re-ID it as a conformant suite artifact and register it, or explicitly move/ignore it according to the artifact contract.

- **[Priority: HIGH]** Replace Task 1's ambiguous recursive negative self-test design. Reinvoking the lint with `ADS_REF=$tmp` will recursively run the same negative self-test unless there is an explicit guard. Specify a reusable `run_asserts` function plus isolated expected-failure handling, or a guard such as `ADS_SKIP_SELF_TEST=1`. Use `trap` for temporary-directory cleanup.

- **[Priority: HIGH]** Strengthen the lint beyond token presence. The proposed checks could pass with disconnected keywords inside the right section while failing to encode FR-1, FR-5, and FR-8. Assert the three applicability decisions, explicit one-line N/A justification requirement, edit-before-create behavior for each deliverable, exact symlink target, and command-shaped verification. Add negative fixtures for misplaced trigger, missing plan command, README reference outside Quick Install, and absent N/A policy—not only a missing heading.

- **[Priority: HIGH]** Task 7 does not satisfy the spec's `/feature-spec` dry-run requirement. Copying files and checking that a relative reference exists does not prove generated Agent Readiness contains the matrix or machine-verifiable gates. Add an actual skill-output fixture/evaluation, or revise the approved spec explicitly. Task 8 must not claim this requirement is covered by the installation simulation.

- **[Priority: MEDIUM]** Define how `/feature-plan` verifies an N/A justification. Task 4 promises validation but only requires the literal token `N/A`; that cannot distinguish a justified N/A from an unsupported one. Require a section-scoped check against the upstream spec or a generated plan instruction that quotes the justification and its source.

- **[Priority: MEDIUM]** Do not mark the handoff and registry entry `Shipped` before release completion. Task 8 contains no public-repo PR/merge or suite-core commit gate even though the spec's Deploy Checklist requires both. Keep status `InReview` until those independent repositories are published, or add an ordered release/finalization task.

- **[Priority: MEDIUM]** Validate claims of Agent Skills conformance against the official validator. The official specification confirms the name, description, 1024-character, relative-reference, and under-500-lines guidance, but also defines `allowed-tools` as a space-separated string. Existing skills use comma-separated values. The plan should either run `skills-ref validate` when available and resolve findings, or clearly label the documented checks as a partial portable subset without claiming full conformance.

- **[Priority: MEDIUM]** Task 7 is not an end-to-end public installation test because it substitutes local `cp` for the documented `curl` commands. At minimum, parse or execute the exact Quick Install command sequence against a controlled URL fixture and verify expected content, not merely file existence. Prefer `curl -fsSL`; current `-sL` can produce a false success on HTTP errors.

- **[Priority: LOW]** Normalize paths relative to their owning repository. Entries such as `spec-plan-ship-suite/tests/...` are ambiguous from a plan stored in the nested product repository. State explicitly which tasks run from suite-core and which run from `products/spec-plan-ship`.

## Considerations
- The progressive-disclosure placement is sound: the normative reference belongs under `feature-spec/references/`, while `feature-plan` remains independently usable.
- Deferring `tests/run.sh` activation until the artifacts exist is a good TDD ordering decision.
- Explicit `cmp` gates correctly compensate for `check-skills-drift.sh` being advisory.
- Section-scoped extraction and checking that the reference trigger is outside generated template fences are appropriate safeguards, provided their negative behavior is tested.
- The mandatory `CLAUDE.md → AGENTS.md` symlink is a project protocol, not a requirement of the public AGENTS.md standard; present it as such in the reference.
