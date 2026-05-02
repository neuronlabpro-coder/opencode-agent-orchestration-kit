# Debugger analysis: iteration-001-sidecar

## 1. Evidencia analizada

### Hechos observados

- `docs/ai/evolution/runs/iteration-001-sidecar/evaluation.md:11-16` registra cuatro escenarios: `/feature` fail; `/scope` pass; `/evolve` pass; sidecars no obligatorios pass.
- `evaluation.md:24-31` atribuye el fail de `/feature` a que el comando no declara localmente la cadena exacta `lead -> designer si aplica -> researcher -> specifier -> developer -> reviewer` y además enumera `researcher` antes de `designer`.
- `commands/feature.md:3` asigna `/feature` al agente `lead`.
- `commands/feature.md:12-23` define un “Flujo obligatorio” con barreras: analizar objetivo, decidir research/diseño, invocar `researcher` si hace falta, invocar `designer` si hace falta, sintetizar antes de `specifier`, luego `developer`, `reviewer` y cierre.
- `commands/feature.md:17-18` enumera `researcher` antes de `designer`.
- `commands/feature.md:27-33` mantiene barreras de dependencia y declara `evaluator`/`debugger` como sidecars opcionales.
- `AGENTS.md:40-47` define el orden normal como `lead`, `designer si aplica`, `researcher si aplica`, `specifier`, `developer`, `reviewer`.
- `AGENTS.md:49-52` declara los flujos base, incluido `lead -> designer si aplica -> researcher -> specifier -> developer -> reviewer`.
- `docs/ai/evolution/README.md:9-15` repite los flujos base preservados y limita `evaluator`, `debugger` y `evolver` a observabilidad concreta o `/evolve`.
- `agents/lead.md:117-128` repite los flujos base y reafirma que la observabilidad no añade fases obligatorias.
- `evaluation.md:95-99` no observa sidecars como fases obligatorias y señala una posible divergencia documental en `/feature`.
- `evaluation.md:100-104` limita la evaluación a inspección estática de archivos; no hubo conversaciones simuladas, rollback ni diff de cambios de harness.
- `evaluation.md:7` y `evaluation.md:92-93` indican que no se aplicaron cambios de comportamiento ni se creó `change_manifest.json`.

### Inferencias separadas de hechos

- La discrepancia parece principalmente documental/de orquestación declarativa: los archivos globales y del lead preservan el flujo base exacto, pero el comando `/feature` puede inducir una interpretación distinta al estar escrito como instrucción local ejecutable.
- El riesgo operativo probable es moderado: aunque `/feature` conserva barreras antes de `specifier`, el orden local `researcher` antes de `designer` puede afectar casos donde diseño debe preceder o condicionar research/spec.
- El fail no demuestra que el harness ejecute siempre el flujo incorrecto; solo demuestra que la declaración local de `/feature` es ambigua o divergente frente al criterio benchmark.
- El cambio candidato, si se pide a `evolver`, debería limitarse al nivel `command`/documentación de flujo de `/feature`, no a agentes sidecar ni a reglas AHE globales.

## 2. Estado

**Listo para evolver con propuesta no aplicada/manifest**, no listo para aplicar cambios directamente.

La evidencia es suficiente para pedir a `evolver` una propuesta acotada porque hay: fallo reproducible por inspección estática, rutas y líneas concretas, root cause localizado en `commands/feature.md`, impacto esperable limitado y predicción verificable. No hay evidencia que justifique cambios en `AGENTS.md`, `agents/lead.md`, `/scope`, `/evolve` o agentes sidecar.

## 3. Patrones de fallo

### Patrón A — Divergencia entre flujo global y comando local

- Globalmente, `AGENTS.md`, `docs/ai/evolution/README.md` y `agents/lead.md` declaran `designer si aplica` antes de `researcher`.
- Localmente, `commands/feature.md` decide ambos y luego lista `researcher` antes de `designer`.
- El patrón no aparece en `/scope` ni `/evolve` según `evaluation.md:33-68`.

### Patrón B — Ambigüedad documental vs. barreras correctas

- `/feature` sí contiene barreras antes de `specifier`, `developer` y `reviewer`.
- El fallo se concentra en la ausencia de la cadena exacta y en el orden local de discovery, no en la eliminación de fases principales ni en sidecars obligatorios.

## 4. Root causes

1. **Root cause confirmado:** `commands/feature.md` no declara localmente el flujo base exacto requerido por el benchmark y usa una enumeración que coloca `researcher` antes de `designer` (`commands/feature.md:17-18`) mientras las fuentes globales colocan `designer si aplica` antes de `researcher` (`AGENTS.md:49-52`, `docs/ai/evolution/README.md:9-15`, `agents/lead.md:117-128`).
2. **Root cause secundario:** el comando mezcla “decidir si hace falta research/diseño” con una secuencia operativa concreta sin explicitar cómo resolver casos donde diseño y research son dependientes o independientes. Esto deja espacio a interpretaciones distintas del flujo base.

## 5. Fixes candidatos por nivel de componente

- **Command (`commands/feature.md`)**: propuesta no aplicada para declarar explícitamente la cadena `lead -> designer si aplica -> researcher -> specifier -> developer -> reviewer` y alinear el orden local con las fuentes globales, preservando las barreras existentes.
- **Workflow/documentación AHE**: no se recomienda cambio con la evidencia actual; `AGENTS.md`, `docs/ai/evolution/README.md` y `agents/lead.md` ya contienen el flujo base esperado.
- **Sidecars (`evaluator`, `debugger`, `evolver`)**: no se recomienda cambio; la evaluación no encontró que sean fases obligatorias del flujo normal.

## 6. Riesgos y regresiones

- **Riesgo de regresión por cambio excesivo:** modificar reglas globales o agentes sidecar podría introducir obligatoriedad no deseada; la evidencia solo apunta a `/feature`.
- **Riesgo de ambigüedad persistente:** si `/feature` conserva `researcher` antes de `designer`, futuras ejecuciones pueden discrepar del flujo base o de benchmarks AHE.
- **Riesgo de sobrecorrección:** forzar siempre diseño antes de research podría ser demasiado rígido si hay casos independientes; cualquier propuesta debería preservar la regla existente de paralelizar solo tareas claramente independientes.
- **Riesgo de evidencia limitada:** la evaluación fue estática; no prueba comportamiento en conversaciones reales con subagentes.

## 7. Atribución de cambios previos

No aplica atribución `keep`/`improve`/`rollback+pivot`: no hay `change_manifest.json` previo en esta iteración según `evaluation.md:92-93`, y no se aplicaron cambios de comportamiento.

## 8. Handoff para specifier/evolver/reviewer

- **Specifier/evolver:** preparar una propuesta no aplicada y manifest acotado a `commands/feature.md`, con predicción: el escenario `/feature` debería pasar al declarar localmente el flujo exacto y eliminar la divergencia de orden.
- **Reviewer:** verificar que cualquier diff futuro toca solo documentación/comando de `/feature` salvo nueva evidencia; comprobar que no convierte sidecars en fases obligatorias.
- **Lead:** decidir si solicita a `evolver` el manifest. La evidencia actual permite propuesta, pero no exige aplicación inmediata.

## Nota explícita de no cambio conductual

Este análisis no modifica archivos de comportamiento del harness. Solo se escribe evidencia en `docs/ai/evolution/runs/iteration-001-sidecar/analysis/overview.md`.
