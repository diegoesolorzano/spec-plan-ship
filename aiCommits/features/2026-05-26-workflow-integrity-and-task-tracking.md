# Integridad del flujo de trabajo y seguimiento de tareas

## Resumen
Refuerza la secuencia obligatoria spec > plan > test-plan > Sprint Goal > implement, agrega seguimiento de tareas en tiempo real durante implementacion, e introduce la seccion Agent Readiness en specs.

## Contexto
El flujo de trabajo tenia fugas donde los skills sugerian saltar directamente a implementacion sin pasar por test-plan o mostrar el Sprint Goal. Ademas, los agentes paralelos no tenian visibilidad del progreso de tareas durante ejecucion. Finalmente, feature-plan referenciaba una seccion "Agent Readiness" en specs que no existia.

## Cambios Principales
- feature-plan Step 8: ya no sugiere implementar directamente; redirige a `/test-plan` (full) o muestra Sprint Goal (lite)
- test-plan Step 9: ya no sugiere implementar directamente; muestra Sprint Goal verbatim y sugiere `/tdd`
- Sprint Goal ahora incluye punteros a artefactos (plan file, test plan file) para agentes sin contexto previo
- feature-plan full mode: crea TaskCreate entries para cada tarea del plan con dependencias via TaskUpdate
- feature-spec template: nueva seccion Agent Readiness (Medium+) con Human Documentation, Agent Documentation, Rules, Skills, Discoverability Check

## Archivos Afectados
- skills/feature-plan/SKILL.md
- skills/test-plan/SKILL.md
- skills/feature-spec/SKILL.md

## Como Usar
El flujo se activa automaticamente al usar los skills en secuencia:
1. `/feature-spec` — ahora incluye Agent Readiness para Medium+
2. `/feature-plan` — en full mode crea tareas rastreables; Step 8 redirige correctamente
3. `/test-plan` — Step 9 muestra Sprint Goal con artifact pointers antes de sugerir `/tdd`
4. `/tdd` — implementacion con test-first

## Breaking Changes
Ninguno. Los cambios refuerzan el flujo existente sin romper compatibilidad.
