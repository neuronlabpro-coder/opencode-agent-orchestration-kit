# Pre-apply debugger approval: chg-1/chg-2 only

## 1. Evidencia analizada

- `pre_apply_evaluation.md`: PA1-PA6, incluyendo baseline `node scripts/check-harness.mjs`, fixtures temporales y contrato estático de `/evolve`.
- `change_manifest.json`: únicamente `chg-1-ahe-run-lifecycle-validator` y `chg-2-evolve-evaluation-only-contract`.
- `analysis/overview.md` y detalles:
  - `analysis/detail/ahe-run-lifecycle-validator.md`
  - `analysis/detail/evolve-evaluation-only-contract.md`
- Historial y atribución previa:
  - `docs/ai/evolution/evolution_history.md`
  - `iteration-003-agent-readable-harness/{evaluation.md,analysis/overview.md,change_manifest.json,change_evaluation.json}`
  - `iteration-004-orchestration-audit/{post_manifest_evaluation.md,change_evaluation.json}`

No se inspeccionaron ni editaron archivos de implementación fuera de esta ruta de análisis. No se modificaron providers, modelos, credenciales, comandos, agentes, scripts ni manifest.

## 2. Estado

**Listo para evolver/developer con alcance acotado**: proceder solo con `chg-1-ahe-run-lifecycle-validator` y `chg-2-evolve-evaluation-only-contract`.

No listo para aplicar `chg-3` ni `chg-4` en este pase: ambos permanecen **fuera de alcance explícito** por aprobación del usuario, aunque el manifest los conserve como propuestas no aplicadas.

## 3. Patrones de fallo confirmados/refinados

### chg-1: lifecycle del validador AHE

Patrón confirmado: el validador local modela todo directorio de `docs/ai/evolution/runs/` como run completo y falla en un estado evaluator-only legítimo.

Evidencia nueva de pre-apply que refina el patrón:

- PA3 confirma baseline actual limpio para el repo completo: `Harness check passed.`
- PA4 reproduce la falla en fixture temporal evaluator-only: `missing analysis/overview.md`.
- PA5 confirma control negativo: un manifest sin análisis/evaluación sigue fallando, por lo que el fix no debe ser una exención amplia de runs incompletos.

### chg-2: contrato `/evolve` evaluation-only/debugger-only

Patrón confirmado: `/evolve` sí gatea evolver por evidencia, pero su resultado esperado conserva wording de manifest obligatorio y no expresa una rama audit/evaluation-only o debugger-only.

Evidencia nueva de pre-apply que refina el patrón:

- PA6 confirma estáticamente que el wording actual contiene `Manifest creado o actualizado`.
- PA6 no encuentra rama explícita `evaluation-only`/`debugger-only` en `commands/evolve.md` + `docs/ai/harness/commands.md`.
- La evidencia es suficiente para aclaración de contrato; comportamiento real requiere replay post-apply antes de marcar `keep`.

## 4. Root causes

- **RC1 confirmado:** `scripts/check-harness.mjs` carece de modelo de lifecycle AHE y por eso confunde fase intermedia evaluator→debugger con run completo inválido.
- **RC2 confirmado con límite:** `commands/evolve.md` describe el cierre happy-path completo como salida esperada sin rama explícita para auditorías evaluation-only/debugger-only. La causa contractual está confirmada; el impacto conductual en agentes queda pendiente de replay.

No se confirmó un nuevo root cause que obligue a cambiar el alcance aprobado.

## 5. Fixes candidatos por nivel de componente

- **Tool (`chg-1`):** ajustar `scripts/check-harness.mjs` para reconocer estados AHE intermedios explícitos, manteniendo fallo para manifest sin `analysis/overview.md`/`change_evaluation.json` según corresponda; documentarlo en `docs/ai/harness/checks.md`.
- **Command/memory (`chg-2`):** aclarar `commands/evolve.md` y `docs/ai/harness/commands.md` para separar flujo AHE completo de ramas evaluation-only/debugger-only con stop conditions y manifest condicionado por evidencia + aprobación de alcance.

No aplicar fixes candidatos para:

- `chg-3-slash-command-contract-coverage`.
- `chg-4-design-open-design-baseurl-contract`.

## 6. Riesgos, regresiones y criterios de falsificación

### chg-1

Riesgos a vigilar:

- El validador acepta runs abandonados o incoherentes demasiado ampliamente.
- La excepción de lifecycle oculta errores de manifest.
- Las reglas del check se vuelven difíciles de entender.

Criterios de falsificación/bloqueo:

- Fixture evaluator-only sigue fallando con `missing analysis/overview.md` sin estado in-progress explícito.
- Fixture con `change_manifest.json` pero sin análisis/evaluación pasa indebidamente.
- Runs completos previos dejan de validar manifests o JSON inválido.

### chg-2

Riesgos a vigilar:

- `/evolve` empieza a omitir manifest en cambios reales del harness.
- El contrato queda tan ramificado que agentes no distinguen flujo completo de auditoría parcial.
- Se debilita la expectativa de full AHE cuando usuario sí autoriza cambio.

Criterios de falsificación/bloqueo:

- Inspección estática post-apply aún muestra manifest como salida incondicional.
- La rama evaluation-only/debugger-only permite saltar manifest cuando hay cambio aplicado o usuario pidió evolución completa.
- Replay autorizado de `/evolve` con “no edits/no manifest” crea o presiona manifest, o un replay full-flow autorizado deja de exigir manifest antes de implementación.

## 7. Atribución histórica

- Iteration 003 `chg-1` (`docs/ai/harness` como fuente de verdad): **keep → refine/improve**, no contradicción. `chg-2` actual usa esa fuente de verdad y añade claridad a un contrato específico; no revierte el diseño.
- Iteration 003 `chg-2` (`scripts/check-harness.mjs`): **keep → improve**, confirmando el risk task previo `validator-blocks-valid-future-run-shape`. `chg-1` actual corrige un riesgo previsto, no invalida los predicted fixes confirmados del validador.
- Iteration 003 `chg-3` (evidencia/replay): **keep con limitación**, sin regresión nueva. La falta de replay para `chg-2` se mantiene explícitamente como limitación y validación post-apply, consistente con no sobredeclarar evidencia.

El `change_evaluation.json` de iteration-004 sigue siendo no-aplicado/improve pending approval; esta aprobación pre-apply no convierte predicciones en `keep`.

## 8. Handoff para evolver/developer/reviewer

Proceder con developer/evolver solo si se mantiene este alcance:

1. Aplicar `chg-1` en `scripts/check-harness.mjs` y `docs/ai/harness/checks.md`.
2. Aplicar `chg-2` en `commands/evolve.md` y `docs/ai/harness/commands.md`.
3. No tocar archivos de `chg-3`/`chg-4` salvo coincidencia inevitable de `docs/ai/harness/commands.md`, y en ese archivo limitar el diff a la sección de `/evolve`.

Validación mínima post-apply antes de reviewer:

1. `node scripts/check-harness.mjs` en repo actual.
2. Fixture evaluator-only: debe pasar o reportar estado in-progress explícito, no `missing analysis/overview.md` como fallo final.
3. Fixture manifest-sin-analysis: debe seguir fallando.
4. Inspección estática: `/evolve` debe documentar ramas full-flow y evaluation-only/debugger-only por separado.
5. Si se autoriza replay: `opencode run --command evolve` con “no edits/no manifest” debe detenerse en handoff sin crear manifest; un escenario full-flow autorizado debe conservar manifest como precondición antes de implementación.

Recomendación: **proceed**, con bloqueo automático si cualquier cambio toca `chg-3`, `chg-4`, providers/modelos/credenciales o intenta marcar predicciones como confirmadas sin validación post-apply.
