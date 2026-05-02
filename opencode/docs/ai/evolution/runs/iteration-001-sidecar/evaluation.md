# Evaluation: iteration-001-sidecar

## 1. Objetivo evaluado

Validar que el harness actual preserva los flujos base y que `evaluator`, `debugger` y `evolver` actúan solo como sidecars, sin convertirse en fases obligatorias del flujo normal.

Alcance solicitado: inspección de archivos de harness bajo `commands/`, `agents/`, `AGENTS.md`, `docs/ai/evolution/` y escritura de evidencia únicamente en esta iteración. No se aplicaron cambios de comportamiento ni se invocó `developer`.

## 2. Escenarios ejecutados

| Escenario | Resultado |
| --- | --- |
| 1. `/feature` declara el flujo `lead -> designer si aplica -> researcher -> specifier -> developer -> reviewer` | fail |
| 2. `/scope` declara `researcher -> specifier` y usa `debugger` solo con trazas/resultados explícitos | pass |
| 3. `/evolve` preserva el flujo AHE completo | pass |
| 4. `evaluator`/`debugger`/`evolver` están registrados pero no son fases obligatorias del flujo normal | pass |

## 3. Resultados por escenario

### Escenario 1 — `/feature` declara el flujo base completo

**Resultado: fail**

Evidencia observada:

- `commands/feature.md:3` define el comando con `agent: lead`.
- `commands/feature.md:12-23` define un “Flujo obligatorio” con barreras: analiza objetivo, decide research/diseño, invoca `researcher` si hace falta, invoca `designer` si hace falta, luego `specifier`, `developer` y `reviewer`.
- `commands/feature.md:33` declara que `evaluator` y `debugger` son sidecars opcionales.
- `AGENTS.md:49-52` y `docs/ai/evolution/README.md:9-12` sí declaran el flujo base exacto `lead -> designer si aplica -> researcher -> specifier -> developer -> reviewer`.

Motivo del fail: el criterio pide verificar que **`/feature`** declare el flujo `lead -> designer si aplica -> researcher -> specifier -> developer -> reviewer`. El archivo `commands/feature.md` declara un flujo secuencial equivalente en términos de barreras principales, pero no contiene esa cadena ni ese orden explícito: invoca `researcher` antes de `designer` en `commands/feature.md:17-18`, mientras el flujo base documentado pone `designer si aplica` antes de `researcher`. Se marca fail por discrepancia/ambigüedad local en el comando, no por ausencia total de barreras.

### Escenario 2 — `/scope` declara `researcher -> specifier` y debugger condicional

**Resultado: pass**

Evidencia observada:

- `commands/scope.md:3` define el comando con `agent: scoper`.
- `commands/scope.md:10` declara explícitamente: “Ejecuta exclusivamente el flujo researcher → specifier.”
- `commands/scope.md:14-18` prohíbe `designer`, `developer`, `reviewer`, implementación y modificación de archivos de aplicación.
- `commands/scope.md:19-24` exige invocar primero a `researcher`, esperar su resultado, usar `debugger` solo con trazas/resultados existentes o evidencia previa concreta, y luego invocar a `specifier`.
- `agents/scoper.md:61-74` repite el flujo obligatorio: researcher, espera, debugger solo si hay trazas/resultados o evidencia concreta, síntesis, y specifier.

### Escenario 3 — `/evolve` preserva flujo AHE completo

**Resultado: pass**

Evidencia observada:

- `commands/evolve.md:10` ordena ejecutar el flujo AHE completo.
- `commands/evolve.md:12-18` define precondiciones: comprobar git, bloquear rollback sin git, leer `docs/ai/evolution/README.md` e identificar iteración objetivo.
- `commands/evolve.md:19-29` declara el flujo obligatorio completo: `evaluator`, `debugger`, `evolver` si hay evidencia, revisión de manifest, `developer` solo si procede/aprobado, nueva medición con `evaluator`, atribución con `debugger`, `reviewer`, y cierre con decisión.
- `docs/ai/evolution/README.md:45-56` documenta el flujo AHE sidecar completo para evolucionar el harness, no para cada feature normal.
- `agents/lead.md:166-180` documenta el flujo AHE explícito para `/evolve` o evolución aprobada del harness.

### Escenario 4 — sidecars registrados pero no obligatorios en flujo normal

**Resultado: pass**

Evidencia observada:

- Registro global: `AGENTS.md:21-24` lista `scoper`, `evaluator`, `debugger` y `evolver`; `evaluator`, `debugger` y `evolver` se describen como sidecars opcionales/exclusivos.
- Flujo normal preservado: `AGENTS.md:40-47` define orden normal `lead`, `designer si aplica`, `researcher si aplica`, `specifier`, `developer`, `reviewer`; `AGENTS.md:49-53` lista los flujos base sin sidecars.
- No obligatoriedad: `AGENTS.md:64-80` dice que la observabilidad no redefine el flujo normal y que `evaluator`, `debugger` y `evolver` son sidecars opcionales con condiciones de uso.
- Lead: `agents/lead.md:58-59` prohíbe insertar `evaluator`, `debugger` o `evolver` como fases obligatorias del flujo normal y prohíbe `evolver` en features normales de app.
- Lead: `agents/lead.md:115-128` reafirma que la observabilidad no añade fases obligatorias y limita sidecars a evidencia concreta o evolución del harness.
- Agentes sidecar: `agents/evaluator.md:31-50`, `agents/debugger.md:23-42` y `agents/evolver.md:23-55` declaran explícitamente que no forman parte del flujo normal de feature o no participan en features normales de apps.

## 4. Comandos o pasos usados

Pasos de inspección:

- Listado de archivos relevantes con glob sobre `commands/**/*`, `agents/**/*` y `docs/ai/evolution/**/*`.
- Lectura de:
  - `AGENTS.md`
  - `commands/feature.md`
  - `commands/scope.md`
  - `commands/evolve.md`
  - `agents/lead.md`
  - `agents/scoper.md`
  - `agents/evaluator.md`
  - `agents/debugger.md`
  - `agents/evolver.md`
  - `docs/ai/evolution/README.md`
  - `docs/ai/evolution/benchmarks/manual-scenarios.md`
- Comando ejecutado para crear el directorio de evidencia:
  - `ls "docs/ai/evolution/runs" && mkdir -p "docs/ai/evolution/runs/iteration-001-sidecar"`

## 5. Evidencia y rutas relevantes

- Evidencia principal escrita en: `docs/ai/evolution/runs/iteration-001-sidecar/evaluation.md`.
- No se creó `change_manifest.json` porque esta evaluación no propone ni aplica cambios de comportamiento; la decisión de manifest queda para `lead`/`evolver` si consideran accionable el fail del escenario 1.

## 6. Regresiones observadas

- No se observaron sidecars como fases obligatorias en el flujo normal.
- Posible regresión/documentación divergente: `/feature` no declara localmente el flujo base exacto y ordena `researcher` antes de `designer`, mientras `AGENTS.md` y `docs/ai/evolution/README.md` documentan `designer si aplica` antes de `researcher`.

## 7. Limitaciones

- La evaluación fue estática por inspección de archivos; no se ejecutaron conversaciones simuladas con subagentes.
- `README.md` raíz no existe en `/path/to/opencode` durante esta ejecución; se usó `docs/ai/evolution/README.md` como README de AHE relevante.
- No se ejecutó rollback ni diff de cambios de harness porque el objetivo prohibía cambios de comportamiento.

## 8. Handoff para debugger/reviewer

- Para `debugger`: analizar si la discrepancia de `/feature` es un fallo real de orquestación o solo una diferencia documental. Evidencia focal: `commands/feature.md:12-23`, `AGENTS.md:49-52`, `docs/ai/evolution/README.md:9-15`, `agents/lead.md:117-128`.
- Para `reviewer`: verificar que este artefacto no modifica archivos de comportamiento del harness y que el fail del escenario 1 está suficientemente justificado antes de pedir un manifest o un cambio.
