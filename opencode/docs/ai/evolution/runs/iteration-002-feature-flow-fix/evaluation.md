# Evaluation: iteration-002-feature-flow-fix

## 1. Objetivo evaluado

Verificar el cambio aplicado en `commands/feature.md` tras el manifest propuesto en `iteration-001-sidecar`.

Cambio evaluado:

- `chg-1`: alinear `/feature` con el flujo base `lead -> designer si aplica -> researcher -> specifier -> developer -> reviewer`.

## 2. Escenarios ejecutados

| Escenario | Resultado |
| --- | --- |
| `/feature` declara localmente el flujo base exacto | pass |
| `/feature` elimina la ambigüedad researcher-before-designer | pass |
| `/feature` no convierte `evaluator`/`debugger` en fases obligatorias | pass |
| `/scope` conserva `researcher -> specifier` con debugger opcional | pass |
| `/evolve` conserva el flujo AHE completo | pass |
| Sidecars siguen registrados pero no obligatorios en flujo normal | pass |

## 3. Evidencia

### `/feature`

- `commands/feature.md` contiene la cadena exacta `lead -> designer si aplica -> researcher -> specifier -> developer -> reviewer`.
- El paso local de diseño aparece antes que research cuando el diseño afecta UX, marca, layout, interacción o criterios de aceptación.
- El comando conserva la excepción: `designer` y `researcher` pueden paralelizarse solo si sus resultados son independientes.
- `evaluator` y `debugger` siguen definidos como sidecars opcionales, no como pasos obligatorios.

### `/scope`

- `commands/scope.md` declara `researcher -> specifier`.
- `debugger` solo aparece como sidecar opcional si el usuario pide analizar trazas/resultados existentes o hay evidencia previa concreta.

### `/evolve`

- `commands/evolve.md` conserva el flujo AHE completo: `evaluator`, `debugger`, `evolver`, manifest, aplicación aprobada, reevaluación, atribución y review.

### Fuentes globales

- `AGENTS.md` y `agents/lead.md` preservan los flujos base y la regla de observabilidad sidecar.

## 4. Limitaciones

- Evaluación estática por inspección de archivos; no ejecuta una conversación real multiagente.
- Suficiente para atribuir el fix documental/comando detectado por `iteration-001-sidecar`, no para validar comportamiento runtime general.

## 5. Resultado

El cambio aplicado confirma los predicted fixes de `chg-1` sin regresiones observadas en `/scope`, `/evolve` ni en la opcionalidad de sidecars.
