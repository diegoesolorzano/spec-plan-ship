# Feature: Agent-docs protocol — forma obligatoria de los entregables Agent Readiness

**Feature ID:** sps-agent-docs-protocol
**Repo:** spec-plan-ship
**Issue:** none
**Upstream:** none
**Date:** 2026-07-25
**Status:** InReview

## Problem Statement

`/feature-spec` ya exige agent docs como entregable para Medium+, pero no define su **forma**: no hay plantilla del `AGENTS.md` local, ni criterio de skill conforme (agentskills.io), ni criterio de regla-ligera, ni el default "editar el artefacto existente antes que crear uno nuevo". `/feature-plan` tiene una task de Agent Documentation con Verify difuso ("a cold agent can find it"). Resultado: cada sesión improvisa la forma y la calidad de los docs varía.

## Background

- Protocolo probado en nodo-ia (auditoría facade + docs, 2026-07-24/25); handoff origen:
  `spec-plan-ship-suite/docs/specs/handoff-agent-docs-protocol.md`.
- Claves probadas: AGENTS.md jerárquico closest-wins con raíz mínima; `CLAUDE.md` SIEMPRE symlink de `AGENTS.md`; plantilla local de 25-60 líneas SIN inventarios de archivos; reglas always-on ligeras con el detalle en la skill (progressive disclosure); enforcement determinista opcional vía gate de CI (ejemplo: `nodo-ia/scripts/checks/check-facade-deep-imports.sh`, bloque `[agents-md]`).
- Alternativa rechazada: acoplar el estándar a nodo-ia — la suite corre en cualquier repo, el estándar debe ser agnóstico.
- Aplica a la capa paga Y pública: es protocolo de documentación, no capacidad paga.

## Requirements

- [ ] FR-1: `/feature-spec` § Agent Readiness da **forma obligatoria** a los 3 entregables Medium+: (a) skill conforme (ver FR-8), (b) `AGENTS.md` local por carpeta de la feature + symlink `CLAUDE.md` que apunta a `AGENTS.md`, (c) regla siempre-ligera (invariantes 1 línea + puntero a la skill; el detalle NUNCA en la regla). La plantilla del AGENTS.md marca 25-60 líneas como **guía de tamaño esperado**, no como gate estricto.
- [ ] FR-2: Los 3 entregables se gatean en el Sprint Goal ("Done when") con evidencia machine-verifiable: existencia del archivo, symlink correcto (`readlink` == `AGENTS.md`), skill dentro del límite de líneas.
- [ ] FR-3: **Default editar-sobre-crear** explícito en los 3 entregables: extender la skill dueña / actualizar el AGENTS.md de la carpeta / actualizar la regla vigente; crear nuevo SOLO cuando es un subsistema, carpeta o constraint genuinamente nuevo.
- [ ] FR-4: La plantilla del AGENTS.md local y los criterios (skill conforme, regla-ligera) viven en `skills/feature-spec/references/agent-docs-standard.md` — agnóstico de repo, cargado con trigger explícito ("lee X SI la feature es Medium+"), NO inflando el body de la skill.
- [ ] FR-5: **Matriz de aplicabilidad determinista** (vive en el estándar, resumida en Agent Readiness): **skill** = obligatoria salvo trivial o cubierta por una existente (nombrarla); **AGENTS.md** = obligatorio si y solo si la feature crea o modifica una carpeta de módulo/facade; **regla** = obligatoria si y solo si introduce un constraint always-on nuevo. Todo "N/A" exige justificación explícita de 1 línea en el spec — nunca se omite en silencio. `/feature-plan` materializa los entregables aplicables (o sus justificaciones N/A) con Verify por comando.
- [ ] FR-6: El gate determinista de CI se documenta como **patrón opcional** en el reference ("si tu repo tiene manifiesto de módulos, cablea el check — ejemplo en nodo-ia"), no como requisito.
- [ ] FR-7: Réplicas byte-idénticas en todas las superficies (feature-spec y feature-plan: `~/.claude/skills`, `~/.agents/skills`, repo público) incluyendo el nuevo `references/`, con filas nuevas en `surfaces.list`. La evidencia de identidad es `cmp` explícito por par de superficies (el `check-skills-drift.sh` actual es advisory — siempre exit 0 — y NO sirve como gate).
- [ ] FR-8: **Subconjunto de conformidad de skill** definido en el estándar: `name` == nombre del directorio (kebab-case); frontmatter YAML válido con `name` y `description`; `description` dice QUÉ hace + CUÁNDO usarla con keywords de disparo (≤1024 chars); body <500 líneas; detalle extenso en `references/` con trigger de carga explícito. Si el repo tiene `skills-ref validate` disponible, usarlo como validador primario; los checks anteriores son el mínimo portable.
- [ ] FR-9: El README público (`spec-plan-ship/README.md`) actualiza su Quick Install para descargar también `skills/feature-spec/references/agent-docs-standard.md` (hoy solo baja `SKILL.md` — una instalación sin el reference dejaría la skill apuntando a un archivo inexistente).
- [ ] FR-10: Behavior test nuevo en suite-core (`tests/behavior/`) que valida forma con asserts conscientes de sección (no greps sueltos): el estándar existe y contiene plantilla + matriz + criterio de skill + criterio de regla-ligera; feature-spec referencia el estándar y el default editar-sobre-crear; feature-plan trae Verify por comando (symlink/existencia); el README instala el reference.

## Acceptance Criteria

- **Given** un spec Medium+, **When** `/feature-spec` redacta Agent Readiness, **Then** aplica la matriz de aplicabilidad: cada entregable aparece con forma obligatoria y gate en el Sprint Goal, o con justificación N/A explícita de 1 línea.
- **Given** la skill feature-spec, **When** una feature es Medium+, **Then** el body instruye leer `references/agent-docs-standard.md` (plantilla AGENTS.md, matriz de aplicabilidad, subconjunto de conformidad de skill, criterio regla-ligera, gate CI opcional).
- **Given** un plan Medium+, **When** `/feature-plan` genera la task de Agent Documentation, **Then** su Verify contiene comandos concretos por entregable aplicable — `test -f {dir}/AGENTS.md`, `[ "$(readlink {dir}/CLAUDE.md)" = "AGENTS.md" ]`, skill <500 líneas — o verifica la justificación N/A.
- **Given** las 3 superficies de cada skill, **When** se comparan con `cmp` par a par (SKILL.md y references), **Then** son byte-idénticas.
- **Given** una instalación pública siguiendo el Quick Install del README, **When** se instala feature-spec, **Then** `references/agent-docs-standard.md` queda instalado junto al SKILL.md.

## Sprint Goal

> `/feature-spec` y `/feature-plan` imponen la forma probada de los agent docs (skill conforme, AGENTS.md local + symlink, regla ligera; editar antes que crear) con un estándar agnóstico en `references/`, replicado sin drift e instalable desde el README público.

### Done when

- [ ] Estándar creado — evidence: `test -f products/spec-plan-ship/skills/feature-spec/references/agent-docs-standard.md` exits 0
- [ ] Forma del estándar y wiring de las skills válidos — evidence: `sh tests/behavior/agent-docs-standard-lint.sh` (suite-core) exits 0 (asserts por sección: plantilla AGENTS.md + matriz de aplicabilidad + criterio skill + criterio regla-ligera en el estándar; trigger de carga y default editar-sobre-crear en feature-spec; Verify por comando en feature-plan; reference en el Quick Install del README)
- [ ] feature-spec/SKILL.md sigue <500 líneas — evidence: `[ "$(wc -l < products/spec-plan-ship/skills/feature-spec/SKILL.md)" -lt 500 ]` exits 0
- [ ] surfaces.list incluye el reference en las 3 superficies — evidence: `[ "$(grep -c 'agent-docs-standard' surfaces.list)" -ge 3 ]` exits 0
- [ ] Réplicas byte-idénticas — evidence: `cmp` exits 0 para cada par: público↔`~/.claude` y público↔`~/.agents` de `feature-spec/SKILL.md`, `feature-spec/references/agent-docs-standard.md` y `feature-plan/SKILL.md`
- [ ] Suite de tests local pasa (incluye el behavior test nuevo) — evidence: `sh tests/run.sh` (suite-core) exits 0
- [ ] Contrato de artefactos conforme — evidence: `scripts/check-artifact-contract.sh products/spec-plan-ship` exits 0

## Scope

### Estándar agnóstico (nuevo)
- [ ] `skills/feature-spec/references/agent-docs-standard.md` — plantilla AGENTS.md local (propósito, superficie pública/facade, invariantes no obvios, gotchas, puntero a la skill dueña; guía de tamaño 25-60 líneas; PROHIBIDO inventarios de archivos y duplicar la skill); matriz de aplicabilidad determinista (FR-5) con exigencia de N/A justificado; subconjunto de conformidad de skill (FR-8); criterio regla-ligera; default editar-sobre-crear; gate CI como patrón opcional con referencia a nodo-ia.

### Skill feature-spec (modificar)
- [ ] `skills/feature-spec/SKILL.md` — § Agent Readiness reescrita: matriz de aplicabilidad resumida, forma obligatoria de los 3 entregables, default editar-sobre-crear, N/A siempre justificado, instrucción de gatearlos en "Done when"; trigger de carga del reference ("SI Medium+, lee references/agent-docs-standard.md"). Body sigue <500 líneas.

### Skill feature-plan (modificar)
- [ ] `skills/feature-plan/SKILL.md` — § Agent Documentation Task: materializa los entregables aplicables del spec (o valida sus justificaciones N/A) con Verify por comando (`test -f {dir}/AGENTS.md`, `[ "$(readlink {dir}/CLAUDE.md)" = "AGENTS.md" ]`, `wc -l` de la skill <500), manteniendo el fallback de evaluación independiente cuando el spec no trae Agent Readiness.

### README público (modificar)
- [ ] `README.md` — Quick Install descarga también `skills/feature-spec/references/agent-docs-standard.md`; la verificación post-install lo incluye.

### Réplicas y registro (suite-core)
- [ ] `~/.claude/skills/feature-spec/` y `~/.agents/skills/feature-spec/` — SKILL.md + `references/` byte-idénticos (cmp).
- [ ] `~/.claude/skills/feature-plan/` y `~/.agents/skills/feature-plan/` — SKILL.md byte-idéntico (cmp).
- [ ] `spec-plan-ship-suite/surfaces.list` — filas nuevas para `feature-spec/references/agent-docs-standard.md` en las 3 superficies.
- [ ] `spec-plan-ship-suite/docs/registry.md` — fila del feature en el índice.
- [ ] `spec-plan-ship-suite/docs/specs/handoff-agent-docs-protocol.md` — Status → Shipped (consumido por este spec).

### Testing
- [ ] `tests/behavior/agent-docs-standard-lint.sh` (suite-core, nuevo) — asserts por sección descritos en FR-10; positivo y negativo (falla si falta una sección requerida). Nota bash 3.2: sin while-read anidado en for (usar awk).
- [ ] `sh tests/run.sh` (suite-core) pasa.
- [ ] `check-artifact-contract.sh products/spec-plan-ship` pasa.
- [ ] `check-skills-drift.sh` se corre como reporte advisory (NO es gate — siempre exit 0); el gate de identidad son los `cmp` del Done when.

## Design Decisions

- **El estándar vive en `references/` de feature-spec (no en la regla ni en el body):** progressive disclosure — solo se paga cuando la feature es Medium+; las reglas quedan ligeras (coherente con `portable-tooling`: recurso bundled dentro de la skill).
- **feature-plan NO referencia el archivo de feature-spec:** evita dependencia cross-skill (una skill no debe asumir que otra está instalada). El plan materializa los checkboxes con comandos Verify autosuficientes; la forma normativa la consumió ya el spec.
- **Default editar-sobre-crear en los 3 entregables:** el anti-patrón real es proliferar skills/reglas/AGENTS.md nuevos que fragmentan la guía; crear nuevo solo ante subsistema/carpeta/constraint genuinamente nuevo (feedback del usuario, 2026-07-25).
- **Matriz de aplicabilidad en vez de "los 3 siempre":** los entregables son obligatorios-por-default pero condicionales por naturaleza (una feature sin carpeta nueva no necesita AGENTS.md); lo no-negociable es que el N/A sea explícito y justificado, nunca silencioso (review r1).
- **25-60 líneas del AGENTS.md = guía, no gate:** un line-count estricto invita a gaming y castiga repos legítimos; el gate duro es existencia + symlink correcto (review r1).
- **`cmp` como gate de identidad, drift script como reporte:** `check-skills-drift.sh` es advisory por diseño (exit 0 siempre); usarlo de gate sería un falso verde (review r1). Endurecer el checker queda como follow-up.
- **Verificación por symlink con `readlink` == `AGENTS.md`:** `test -L` solo acepta que exista un symlink, aunque esté roto o apunte a otra cosa (review r1).
- **Gate CI opcional, no requisito:** no todo consumidor tiene infraestructura de gates; se documenta como patrón con ejemplo en nodo-ia.
- **Ambas capas, mismo texto:** protocolo de documentación, no capacidad paga — el diferencial pago (cross-model reviews) no cambia.

## Out of Scope

- Evals de skills (decidido 2026-07-25: fase posterior).
- Cambios en nodo-ia o su gate de CI (ya existen; solo se referencian como ejemplo).
- Back-migración de specs/planes ya escritos.
- Un checker determinista nuevo en suite-core que valide AGENTS.md/symlinks en repos consumidores (posible follow-up).
- Endurecer `check-skills-drift.sh` para que falle en drift/superficie faltante (follow-up; hoy es advisory por diseño).
- La regla global del usuario `~/.claude/rules/agent-docs-on-features.md` — es config personal fuera del versionado de la suite (violaría `portable-tooling` tratarla como entregable); el usuario la alinea aparte si lo desea.

## Agent Readiness (Medium+ only) — MANDATORY deliverable

- **README.md:** SÍ — el Quick Install debe instalar el `references/` nuevo (FR-9); sin eso la instalación pública queda rota.
- **AGENTS.md:** N/A — no se crea carpeta nueva de módulo; el cambio vive dentro de skills existentes.
- **Runbook:** N/A — el procedimiento replicar-superficies ya está cubierto por `surfaces.list` + los `cmp` del Done when.
- **Rules:** N/A dentro de la suite — no introduce constraint always-on nuevo en los repos de la suite; el criterio regla-ligera queda documentado en el estándar. (La regla global del usuario es personal — ver Out of Scope.)
- **Skills:** este feature ES la extensión de las skills dueñas (feature-spec, feature-plan) — no se crea skill nueva, coherente con el propio default que instaura.
- **Discoverability:** un agente frío que corra `/feature-spec` en cualquier repo llega al estándar vía el trigger del body; una instalación pública desde el README trae el reference. ✔

## Risks

- **Riesgo:** inflar `feature-spec/SKILL.md` y romper su propio criterio <500 líneas. **Mitigación:** el detalle va al reference; el body solo gana el trigger + forma resumida; gate `wc -l` en Done when. **Rollback:** revert del commit.
- **Riesgo:** drift entre superficies al agregar `references/`. **Mitigación:** filas en `surfaces.list` en el mismo cambio; gate `cmp` por par en Done when; drift script como reporte. **Rollback:** revert + re-sync.
- **Riesgo:** instalaciones públicas viejas (solo SKILL.md) referencian un archivo inexistente. **Mitigación:** el trigger en el body indica el path relativo dentro de la skill; README actualizado; el fallo es visible (archivo no encontrado), no silencioso. **Rollback:** revert del README.
- **Riesgo:** la plantilla resulta demasiado rígida para repos chicos. **Mitigación:** tamaño como guía (no gate); matriz de aplicabilidad permite N/A justificado; el gate CI es opcional.

## Deploy Checklist

- [ ] Tests: `sh tests/run.sh` (incluye `agent-docs-standard-lint.sh`), `check-artifact-contract.sh products/spec-plan-ship`, `cmp` de réplicas — todo en verde; `check-skills-drift.sh` como reporte.
- [ ] Deploy: PR en `spec-plan-ship` (público) + commit en suite-core (surfaces.list, registry, handoff Status, behavior test) + sync réplicas `~/.claude` y `~/.agents`.
- [ ] Verification: dry-run de `/feature-spec` sobre un repo fixture desechable en el scratchpad de la sesión (no un repo real): el output de Agent Readiness muestra la matriz y el trigger del estándar; se borra el fixture al final.
- [ ] Pending: checker determinista de AGENTS.md/symlink en suite-core y endurecimiento de `check-skills-drift.sh` (follow-ups opcionales).

## Open Questions

Ninguna.

## Review Notes

- 2026-07-25 · spec-reviewer · round 1 · REQUEST_CHANGES · incorporate-and-stop · docs/specs/sps-agent-docs-protocol-r1.md
