# Protocolo de testing funcional operativo

## Resumen

Se agrega una metodologia publica para definir escenarios funcionales operativos antes de enviar features Medium+ que cambian flujos reales de producto.

## Contexto

El flujo anterior cubria especificacion, plan, test plan tecnico y TDD, pero no obligaba a probar explicitamente que el comportamiento operativo completo funcionara en condiciones realistas. Este cambio incorpora un paso dedicado para validar actores, estado persistido, efectos visibles, side effects y criterios de ship gate.

La aplicacion pagada de este protocolo queda fuera de este cambio y se mantiene como trabajo separado en `spec-plan-ship-suite` issue #6.

## Cambios Principales

- Agregar el skill `/operational-test-plan` para diseñar escenarios operativos y gates de envio
- Agregar la regla de testing funcional operativo para features Medium+ con impacto en workflows reales
- Agregar la regla de alcance documental para decidir entre README, docs, reglas, skills, roadmap y changelog
- Actualizar el flujo publico a `spec -> plan -> test-plan -> operational-test-plan -> TDD -> operational validation -> ship`
- Actualizar README y ROADMAP para documentar la metodologia y el trabajo pendiente de enforcement pagado
- Ajustar los skills de plan y test plan para integrar el nuevo paso operativo cuando aplique

## Archivos Afectados

- `README.md`
- `ROADMAP.md`
- `rules/feature-workflow.md`
- `rules/operational-functional-testing.md`
- `rules/documentation-scope.md`
- `skills/feature-plan/SKILL.md`
- `skills/test-plan/SKILL.md`
- `skills/operational-test-plan/SKILL.md`

## Cómo Usar

Para features Medium+ que cambian workflows, automatizaciones, agentes, handoffs, webhooks, colas, crons, integraciones externas o journeys multi-paso:

1. Crear spec con `/feature-spec`
2. Crear plan con `/feature-plan`
3. Crear test plan tecnico con `/test-plan`
4. Crear validacion operativa con `/operational-test-plan`
5. Implementar con TDD
6. Ejecutar los escenarios bloqueantes del operational test plan antes de marcar el feature como enviado

## Breaking Changes

No hay breaking changes de API o formato de artefactos existentes. El cambio agrega una expectativa metodologica para features Medium+ con comportamiento operativo.
