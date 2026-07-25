# Plan: Agent-docs protocol — forma obligatoria de los entregables Agent Readiness

**Feature ID:** sps-agent-docs-protocol
**Repo:** spec-plan-ship
**Issue:** none
**Upstream:** docs/specs/sps-agent-docs-protocol.md
**Date:** 2026-07-25
**Status:** Shipped

## Overview

Cambio docs-only sobre 2 skills + 1 reference nuevo + README, con gate de forma vía un behavior test POSIX sh en suite-core y réplicas byte-idénticas a `~/.claude` y `~/.agents`. TDD: el lint se escribe primero (RED por invocación directa), los artefactos lo ponen en verde; la activación en `run.sh` llega al final para no dejar la suite roja durante tareas intermedias.

**Idioma:** `agent-docs-standard.md` y todos los cambios de skills/README se redactan en **inglés** (idioma del repo público). Los headings del contrato de abajo son **literales, no traducibles** — el lint los busca tal cual.

**Working dirs (normalización de paths):** Tasks 0, 1, 6 (surfaces/run.sh), 8 y 9 corren desde **suite-core** (`spec-plan-ship-suite/`); Tasks 2-5 y 7 desde **`products/spec-plan-ship/`**. En cada task los paths son relativos a su working dir declarado.

## Sprint Goal

_Quoted from the spec's `## Sprint Goal` (authored in `/feature-spec`, not here). Reproduce it verbatim as a reference target._

> `/feature-spec` y `/feature-plan` imponen la forma probada de los agent docs (skill conforme, AGENTS.md local + symlink, regla ligera; editar antes que crear) con un estándar agnóstico en `references/`, replicado sin drift e instalable desde el README público.

### Done when

- [ ] Estándar creado — evidence: `test -f products/spec-plan-ship/skills/feature-spec/references/agent-docs-standard.md` exits 0
- [ ] Forma del estándar y wiring de las skills válidos — evidence: `sh tests/behavior/agent-docs-standard-lint.sh` exits 0
- [ ] feature-spec/SKILL.md sigue <500 líneas — evidence: `[ "$(wc -l < products/spec-plan-ship/skills/feature-spec/SKILL.md)" -lt 500 ]` exits 0
- [ ] surfaces.list incluye el reference en las 3 superficies — evidence: `[ "$(grep -c '^feature-spec-ref' surfaces.list)" -ge 3 ]` exits 0
- [ ] Réplicas byte-idénticas — evidence: `cmp` exits 0 por par (público↔`~/.claude`, público↔`~/.agents`) para feature-spec/SKILL.md, references/agent-docs-standard.md y feature-plan/SKILL.md
- [ ] Suite de tests local pasa — evidence: `sh tests/run.sh` exits 0
- [ ] Contrato de artefactos conforme — evidence: `scripts/check-artifact-contract.sh products/spec-plan-ship` exits 0

## Shared Types

No aplica código; el "contrato" compartido son los asserts del lint (Task 1), **section-scoped** (rango awk entre headings) y **conscientes de estructura** — no tokens sueltos:

```
agent-docs-standard.md — headings literales requeridos (inglés):
  "## Applicability matrix" · "## Edit-before-create default" · "## AGENTS.md template"
  · "## Skill conformance" · "## Lightweight-rule criterion" · "## Optional CI gate"
Dentro de "## Applicability matrix": las 3 decisiones ("skill", "AGENTS.md", "rule")
  y la política N/A ("N/A" + "justification").
Dentro de "## AGENTS.md template": "CLAUDE.md", "symlink",
  la verificación exacta `readlink` == "AGENTS.md", y la prohibición ("file inventories").
Dentro de "## Skill conformance": "500", "1024", "kebab-case",
  y la etiqueta de subconjunto ("portable subset").
Dentro de "## Edit-before-create default": "extend" y "create".

feature-spec/SKILL.md:
  - trigger "references/agent-docs-standard.md" FUERA de los fences ``` (awk toggle)
  - "edit-before-create"/"Edit-before-create" en el body (proceso)
  - dentro del fence de § Agent Readiness: "N/A" + "justification" y las 3 filas de la matriz

feature-plan/SKILL.md — dentro de § "Agent Documentation Task":
  "test -f" · la verificación `readlink` == "AGENTS.md" · "wc -l" · "500" ·
  "N/A" + "quote" (la instrucción de citar la justificación y su fuente del spec)

README.md — dentro del rango de "Quick Install" (hasta el siguiente heading):
  "references/agent-docs-standard.md"
```

## Tasks

### Task 0: Conformar el handoff de suite-core (baseline verde)
- **Working dir:** suite-core
- **Files:** `docs/specs/handoff-agent-docs-protocol.md` → renombrar a `docs/specs/suite-agent-docs-handoff.md` (Feature ID → `suite-agent-docs-handoff`), `docs/registry.md` (fila del handoff)
- **Do:** El artefacto viola el contrato HOY (`check-artifact-contract.sh .` exit 1: id sin prefijo `suite-`). Renombrar archivo + Feature ID, registrar en el índice. Status queda `Draft` hasta la release (Task 9).
- **Verify:** `scripts/check-artifact-contract.sh .` exits 0.
- **Tests:** cubierto por el checker.
- **Depends on:** None

### Task 1: Behavior test `agent-docs-standard-lint.sh` (RED)
- **Working dir:** suite-core
- **Files:** `tests/behavior/agent-docs-standard-lint.sh` (create)
- **Do:** Test POSIX sh (patrón `triage-skill-lint.sh`): resuelve `ROOT` relativo a sí mismo; overrides herméticos por archivo vía env (`ADS_REF`, `ADS_SPEC_SKILL`, `ADS_PLAN_SKILL`, `ADS_README` — default: paths reales del repo público). **Estructura anti-recursión:** los asserts viven en una función `run_asserts` (o bloque re-ejecutable); el self-test negativo corre SOLO cuando `ADS_SELF_TEST` no está seteado, y las re-invocaciones internas exportan `ADS_SELF_TEST=1`. Cleanup de tmp con `trap ... EXIT`. Asserts según Shared Types (section-scoped con awk; trigger-outside-fence con awk toggle de fences) + `[ "$(wc -l < $ADS_SPEC_SKILL)" -lt 500 ]`. **Self-test negativo (4 fixtures tmp mutados):** (a) reference sin un heading requerido; (b) SKILL.md de feature-spec con el trigger movido DENTRO de un fence; (c) feature-plan sin `test -f`; (d) README con el reference solo fuera de Quick Install — cada uno debe hacer fallar la re-invocación. Cada assert imprime ok/FAIL; exit 1 si alguno falla. **Nota bash 3.2:** sin while-read anidado en for.
- **Verify:** `sh tests/behavior/agent-docs-standard-lint.sh` sale ≠0 AHORA (RED: reference inexistente); `sh tests/run.sh` sigue verde (aún no activado).
- **Tests:** este ES el test (incluye self-test negativo automatizado).
- **Depends on:** None

### Task 2: Estándar `references/agent-docs-standard.md` (la task grande del feature — no es de 2-5 min)
- **Working dir:** products/spec-plan-ship
- **Files:** `skills/feature-spec/references/agent-docs-standard.md` (create)
- **Do:** Documento **en inglés**, agnóstico de repo, con los headings literales del contrato: **## Applicability matrix** (skill = always unless trivial/covered — name which one; AGENTS.md = iff the feature creates/modifies a module/facade folder; rule = iff it introduces a new always-on constraint; every N/A requires an explicit 1-line justification in the spec — never silent); **## Edit-before-create default** (extend the owning skill / update the folder's AGENTS.md / update the existing rule; create new ONLY for a genuinely new subsystem/folder/constraint); **## AGENTS.md template** (1-line purpose · public surface/facade ("external consumers import ONLY from here") · non-obvious local invariants · gotchas · pointer to the owning skill; 25-60 lines as guidance NOT a gate; FORBIDDEN: file inventories, duplicating the skill; the `CLAUDE.md` symlink is **this workflow's protocol** — not part of the public agents.md standard, presented as such — verified with `[ "$(readlink CLAUDE.md)" = "AGENTS.md" ]`); **## Skill conformance** (labeled a **portable subset** of agentskills.io — NOT full conformance: name==dir kebab-case; valid YAML frontmatter with name+description; description WHAT+WHEN with trigger keywords ≤1024 chars; body <500 lines; bulk detail in `references/` with explicit load trigger "read X IF Y"; run `skills-ref validate` as primary validator when available and resolve its findings — note: the official spec defines `allowed-tools` as space-separated; some existing skills use commas, a known divergence to resolve when validating); **## Lightweight-rule criterion** (1-line invariants + pointer; detail NEVER in the rule); **## Optional CI gate** (pattern: if the repo has a module manifest, wire an AGENTS.md+symlink check; example: nodo-ia `scripts/checks/check-facade-deep-imports.sh`, `[agents-md]` block). Sources: agents.md, agentskills.io/specification.
- **Integrates with:** cargado por el body de feature-spec (Task 3).
- **Verify:** asserts del lint sobre el reference pasan (invocación directa desde suite-core).
- **Tests:** cubierto por Task 1.
- **Depends on:** Task 1

### Task 3: feature-spec SKILL.md — trigger en el body, forma en el template
- **Working dir:** products/spec-plan-ship
- **Files:** `skills/feature-spec/SKILL.md` (modify)
- **Do:** **Dos placements distintos — no confundirlos (crítico):** (a) **Body/process:** en Step 3 agregar "If the feature is Medium+, read `references/agent-docs-standard.md` (bundled with this skill) for the normative Agent Readiness templates and criteria" + el default edit-before-create como guideline de proceso. El trigger vive FUERA de los fences del template — los specs generados NO heredan un path que no existe en el repo del usuario. (b) **Dentro del fence del template** (§ Agent Readiness que los specs copian): matriz de aplicabilidad resumida (3 filas de 1 línea: skill/AGENTS.md/rule), exigencia de N/A con justificación explícita de 1 línea, e instrucción de gatear los entregables aplicables en el "Done when" con evidencia por comando (existencia, `readlink` == `AGENTS.md`, `wc -l` <500) — autosuficiente, sin referenciar el archivo del estándar. Mantener README/Runbook/Discoverability. Body <500 líneas.
- **Verify:** lint pasa asserts de feature-spec (trigger fuera de fence, edit-before-create, N/A+justification, filas de matriz); `wc -l` < 500.
- **Tests:** cubierto por Task 1 (incluye negativo b: trigger dentro de fence).
- **Depends on:** Task 2

### Task 4: feature-plan SKILL.md — Agent Documentation Task con Verify por comando
- **Working dir:** products/spec-plan-ship
- **Files:** `skills/feature-plan/SKILL.md` (modify)
- **Do:** Reescribir § "Agent Documentation Task" (Guidelines): la task materializa los entregables **aplicables** del spec con Verify por comando por entregable: `test -f {dir}/AGENTS.md`, `[ "$(readlink {dir}/CLAUDE.md)" = "AGENTS.md" ]`, `[ "$(wc -l < .claude/skills/{name}/SKILL.md)" -lt 500 ]`; default edit-before-create. **Validación N/A:** para cada entregable marcado N/A en el spec, la task generada debe **quote** la justificación de 1 línea y su fuente (la línea del spec) — un N/A sin justificación citada es un gap que la task debe marcar, no aceptar. Mantener el fallback de evaluación independiente cuando el spec no trae Agent Readiness. Sin referencia al archivo de feature-spec (sin dependencia cross-skill).
- **Verify:** lint pasa asserts de feature-plan (`test -f`, `readlink`, `wc -l`/500, N/A+quote).
- **Tests:** cubierto por Task 1 (incluye negativo c).
- **Depends on:** Task 2

### Task 5: README Quick Install + Verify
- **Working dir:** products/spec-plan-ship
- **Files:** `README.md` (modify)
- **Do:** En **Quick Install** (no en Verify/Manual): `mkdir -p .claude/skills/feature-spec/references` + curl del reference con **`curl -fsSL`** (falla en HTTP error — a diferencia del `-sL` existente; alinear las líneas existentes a `-fsSL` es un cambio de 1 token por línea: hacerlo en el mismo edit, es estrictamente mejor). En § Verify, mencionar el reference. Manual Install ya cubre (cp -r).
- **Verify:** lint pasa el assert section-scoped de Quick Install (incluye negativo d).
- **Tests:** cubierto por Task 1.
- **Depends on:** Task 2

### Task 6: Réplicas + surfaces.list + activación en run.sh
- **Working dir:** suite-core (cp hacia `~`)
- **Files:** `~/.claude/skills/feature-spec/` y `~/.agents/skills/feature-spec/` (SKILL.md + `references/`), `~/.claude/skills/feature-plan/SKILL.md`, `~/.agents/skills/feature-plan/SKILL.md`, `surfaces.list` (3 filas grupo `feature-spec-ref`), `tests/run.sh` (línea de activación)
- **Do:** Copiar los 3 archivos del repo público a ambas replicas (cp, byte-idéntico). Agregar a `surfaces.list` el grupo `feature-spec-ref` con sus 3 superficies. Activar el lint en `run.sh`: `[ -f "$ROOT/tests/behavior/agent-docs-standard-lint.sh" ] && assert_exit "agent-docs standard lint" 0 sh "$ROOT/tests/behavior/agent-docs-standard-lint.sh"`.
- **Verify:** los 6 `cmp` del Done when salen 0; `[ "$(grep -c '^feature-spec-ref' surfaces.list)" -ge 3 ]`; `sh tests/run.sh` verde; `scripts/check-skills-drift.sh` (advisory) sin drift nuevo.
- **Depends on:** Task 3, Task 4, Task 5

### Task 7: Simulación de instalación (install-shape test — NO cubre el dry-run del spec)
- **Working dir:** products/spec-plan-ship (fixture en el scratchpad, con cleanup vía trap)
- **Do:** **Parsear los comandos del Quick Install real del README** (extraer los pares URL→destino de las líneas curl con awk/sed) y reproducirlos con `cp` desde el working tree local (mapeando la URL raw al path del repo — sin red). Verificar: cada destino existe, el reference quedó en `.claude/skills/feature-spec/references/`, y el trigger del SKILL.md instalado referencia un path que existe relativo a la skill instalada. Borrar el fixture. **Etiqueta honesta:** esto valida la *forma* de la instalación (paths del README completos y consistentes), no la red ni GitHub.
- **Verify:** todos los checks exit 0; fixture eliminado.
- **Tests:** este ES el install-shape test (efímero, no versionado).
- **Depends on:** Task 6

### Task 8: Dry-run real de `/feature-spec` (verificación del Deploy Checklist)
- **Working dir:** scratchpad (repo git desechable)
- **Do:** Crear un fixture repo mínimo en el scratchpad; correr el flujo de `/feature-spec` (el agente ejecuta la skill actualizada) sobre una mini-feature Medium+ inventada; **evaluar el output generado**: la § Agent Readiness del spec generado contiene la matriz (3 decisiones), N/A justificado si aplica, y gates por comando en su Done when. Registrar el resultado (ok/gaps) y borrar el fixture. Esta verificación es del output del skill (juicio + checks), separada del install-shape test de Task 7.
- **Verify:** el spec generado exhibe matriz + N/A justificado + gates por comando; fixture eliminado.
- **Depends on:** Task 6

### Task 9: Release + registro (cierre)
- **Working dir:** suite-core + products/spec-plan-ship
- **Files:** `docs/registry.md` (fila del feature), `docs/specs/suite-agent-docs-handoff.md` (Status), artefactos del feature (Status)
- **Do:** Batería completa final (`sh tests/run.sh`, `scripts/check-artifact-contract.sh products/spec-plan-ship`, `scripts/check-artifact-contract.sh .`, los 6 `cmp`). Luego release ordenada: rama + commits atómicos + PR en `spec-plan-ship` (público); commits en suite-core (test, surfaces, registry, handoff). **Solo tras el merge del PR público:** handoff → `Shipped`, artefactos del feature → `Shipped`, fila del registry con el PR. Antes de eso todo queda `InReview`.
- **Verify:** todos los comandos del Done when en verde; PR mergeado antes de cualquier `Shipped`.
- **Depends on:** Task 7, Task 8

## Test Matrix

### Shared Test Infrastructure
- **Framework:** POSIX sh (harness `tests/run.sh`, patrón ok/FAIL + `assert_exit`)
- **Fixtures:** ninguno versionado — el lint corre sobre archivos reales con overrides env (`ADS_*`); self-test negativo con copias tmp mutadas + `trap` cleanup
- **Factories:** N/A

### Acceptance Criteria → Test Mapping

| AC | Test Location | Case |
|----|--------------|------|
| Agent Readiness con matriz + forma + N/A justificado | `tests/behavior/agent-docs-standard-lint.sh` | asserts section-scoped feature-spec |
| Body instruye leer el estándar (trigger fuera de fence) | ídem | assert trigger-outside-fence + negativo b |
| feature-plan Verify por comando + N/A quote | ídem | asserts `test -f`/`readlink`/`wc -l`/N-A+quote + negativo c |
| Superficies byte-idénticas | comandos `cmp` (Done when, Task 6) | 6 pares |
| Instalación pública trae el reference | lint (Quick Install scoped + negativo d) + Task 7 | section-scoped + install-shape |
| Output real del skill conforme | Task 8 (dry-run en fixture) | evaluación del spec generado |

### Per-Task Test Cases

#### Task 1: agent-docs-standard-lint.sh
**File:** `tests/behavior/agent-docs-standard-lint.sh`
**Type:** Behavior/lint

| # | Case | Given | When | Then | Type |
|---|------|-------|------|------|------|
| 1 | RED inicial | reference inexistente | invocación directa | exit ≠0 | negativo |
| 2 | GREEN final | Tasks 2-5 hechas | invocación directa | exit 0, todos ok | positivo |
| 3 | heading faltante | tmp sin un heading, `ADS_REF=$tmp ADS_SELF_TEST=1` | re-invocación interna | falla, self-test ok | negativo |
| 4 | trigger dentro de fence | tmp de SKILL.md mutado | re-invocación interna | falla, self-test ok | negativo |
| 5 | feature-plan sin `test -f` | tmp mutado | re-invocación interna | falla, self-test ok | negativo |
| 6 | reference fuera de Quick Install | tmp de README mutado | re-invocación interna | falla, self-test ok | negativo |

### E2E Flows

1. Task 7: install-shape (parsear Quick Install real → cp → checks) → cleanup.
2. Task 8: dry-run de `/feature-spec` en fixture repo → spec generado conforme → cleanup.

## Patterns to Reuse

- `tests/behavior/triage-skill-lint.sh` — lint autosuficiente + override env para hermeticidad.
- `tests/run.sh` — activación guardada `[ -f ... ] && assert_exit`.
- `.claude/rules/portable-tooling.md` — reference bundled dentro de la skill.
- Memoria `bash32-while-read-in-for` — evitar while-read anidado en for.

## Risks

- Lint acoplado a headings exactos → headings literales en inglés fijados aquí; cambiarlos exige tocar lint + estándar juntos (drift visible al correr la suite).
- Trigger dentro del fence del template por error → specs generados con path roto; mitigado con assert + negativo b.
- Recursión del self-test → guard `ADS_SELF_TEST` + `run_asserts`; trap para cleanup.
- Réplicas olvidadas → surfaces.list + cmp en Done when.
- `Shipped` prematuro → Task 9 lo condiciona al merge del PR público.

## Review Notes

- 2026-07-25 · Plan subagent (adversarial) · 3C/4W/3S incorporados: trigger body-vs-template explícito (Task 3), contrato de asserts ampliado y section-scoped (FR-10), assert discriminante para feature-plan (`test -f`/`wc -l`), overrides herméticos + negativo automatizado, activación run.sh diferida a Task 6, E2E install-sim como Task 7, Task 6 depende de Task 5, grep de surfaces anclado a `^feature-spec-ref`, idioma/headings literales declarados, Task 2 sin pretensión 2-5 min.
- 2026-07-25 · plan-reviewer (cmrl) · round 1 · MAJOR_REWORK · incorporate-and-stop · docs/specs/sps-agent-docs-protocol-r2.md — 9 findings: Task 0 nuevo (handoff no conforme, checker rojo hoy); anti-recursión `ADS_SELF_TEST` + `run_asserts` + trap; asserts estructurales (3 decisiones, N/A policy, edit-before-create, readlink exacto) + 4 fixtures negativos; Task 8 nuevo (dry-run real, Task 7 re-etiquetado install-shape y parsea el Quick Install real); N/A verificado por quote de justificación + fuente; `Shipped` solo tras merge (Task 9 release); Skill conformance como "portable subset" + nota `allowed-tools` space-separated; `curl -fsSL`; working dirs normalizados.
