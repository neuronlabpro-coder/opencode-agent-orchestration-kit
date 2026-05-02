# Evaluation: iteration-004-orchestration-audit

## 1. Objetivo evaluado

Auditar la orquestación del harness OpenCode para encontrar mejoras posibles sin
aplicar cambios de implementación. Esta evaluación es el paso `evaluator` de AHE
y se limita a evidencia observable bajo `docs/ai/evolution/runs/iteration-004-orchestration-audit/`.

Fuentes inspeccionadas:

- `AGENTS.md:6-12`, `AGENTS.md:29-35`, `AGENTS.md:63-74`
- `docs/ai/harness/agents.md:3-25`
- `docs/ai/harness/commands.md:3-66`
- `docs/ai/harness/evidence.md:3-42`
- `docs/ai/harness/checks.md:3-31`
- `docs/ai/evolution/benchmarks/manual-scenarios.md:1-123`
- `commands/feature.md:12-35`
- `commands/evolve.md:19-46`
- `commands/design.md:11-32`
- `commands/scope.md:10-26`
- `commands/review.md:7-17`
- `commands/research.md:1-11`, `commands/spec.md:1-11`, `commands/implement.md:1-10`
- `agents/lead.md:36-80`, `agents/lead.md:135-148`, `agents/lead.md:206-233`
- `agents/evaluator.md:31-69`, `agents/debugger.md:23-63`, `agents/evolver.md:23-70`
- `docs/ai/evolution/runs/iteration-001-sidecar/change_manifest.json:21-49`
- `docs/ai/evolution/runs/iteration-002-feature-flow-fix/change_evaluation.json:1-31`
- `docs/ai/evolution/runs/iteration-003-agent-readable-harness/change_manifest.json:21-79`
- `docs/ai/evolution/runs/iteration-003-agent-readable-harness/change_evaluation.json:1-65`

## 2. Escenarios ejecutados

| ID | Escenario | Evidencia | Resultado |
| --- | --- | --- | --- |
| S1 | `harness-structural-check` | `static_contract` + comando local | fail |
| S2 | `feature-flow-sidecar-boundaries` | `static_contract` | pass |
| S3 | `ahe-evaluation-only-safe-path` | `static_contract` | fail |
| S4 | `slash-command-contract-completeness` | `static_contract` | fail |
| S5 | `design-open-design-tool-contract` | `static_contract` | fail |
| S6 | `transcript-replay-orchestration-prompts` | replay definido, no ejecutado | not_run |
| S7 | `previous-iteration-predictions-risk-tasks` | `static_contract` | pass |

Conteo: `pass=2`, `fail=4`, `not_run=1`.

## 3. Resultados por escenario

### S1: `harness-structural-check`

Objetivo: comprobar que las invariantes mecánicas existentes del harness siguen
pasando y que el check tolera una iteración AHE legítimamente en progreso en fase
`evaluator` antes de que `debugger` cree `analysis/overview.md`.

Comando ejecutado antes de crear este artefacto:

```bash
node scripts/check-harness.mjs
```

Resultado observado inicial: `pass`.

Salida relevante:

```text
Harness check passed.
```

Evidencia adicional:

- `docs/ai/harness/checks.md:5-19` declara este check y qué valida.
- El check cubre `default_agent: developer`, frontmatter, contrato local de
  `/feature`, docs principales, benchmark y manifests AHE parseables.

Limitación: el check no cubre todos los hallazgos de esta auditoría; por ejemplo,
no valida que `docs/ai/harness/commands.md` enumere todos los slash commands
existentes ni que las instrucciones de herramientas coincidan con schemas reales.

Comando de verificación ejecutado después de crear
`docs/ai/evolution/runs/iteration-004-orchestration-audit/evaluation.md`:

```bash
node scripts/check-harness.mjs
```

Resultado observado posterior: `fail`.

Salida relevante:

```text
Harness check failed:
- docs/ai/evolution/runs/iteration-004-orchestration-audit: missing analysis/overview.md
```

Resultado: `fail` para la auditoría de orquestación. El check mecánico parece
tratar una iteración AHE recién iniciada por `evaluator` como incompleta/fallida
antes de que el flujo llegue a `debugger`, aunque `agents/evaluator.md:63-69`
recomienda crear solo `evaluation.md` en esta fase y reserva `analysis/` para
`debugger`. Esto puede bloquear validaciones intermedias legítimas de AHE.

### S2: `feature-flow-sidecar-boundaries`

Objetivo: comprobar que la orquestación normal de `/feature` mantiene barreras y
no vuelve obligatorios a `evaluator`, `debugger` o `evolver`.

Esperado:

- `/feature` declara `lead -> designer si aplica -> researcher -> specifier -> developer -> reviewer`.
- `developer` espera criterios de aceptación.
- `reviewer` espera diff revisable.
- Sidecars siguen opcionales salvo evidencia explícita o AHE.

Observado:

- `AGENTS.md:31-35` declara los flujos base y AHE separados.
- `docs/ai/harness/commands.md:13-27` declara el contrato de `/feature` con
  designer antes de spec si aplica, researcher antes de spec si hay incertidumbre,
  paralelización solo si independiente, developer con criterios y reviewer con diff.
- `commands/feature.md:15-24` replica el flujo base y las barreras.
- `commands/feature.md:34-35` mantiene sidecars opcionales y eleva cambios de
  harness a evolución AHE con manifest.
- `agents/lead.md:71-80` contiene las mismas barreras críticas.

Resultado: `pass`.

### S3: `ahe-evaluation-only-safe-path`

Objetivo: comprobar que `/evolve` soporta un camino seguro de evaluación-only
cuando el usuario pide auditoría/evidencia pero prohíbe implementación y manifest.

Esperado:

- El flujo debe permitir que `evaluator` produzca evidencia y entregue handoff a
  `debugger` sin forzar manifest si todavía no hay root cause/predicted fixes o
  si el usuario explícitamente dice “no crear manifest”.
- `evolver` debe crear manifest solo si hay evidencia suficiente y la fase procede.

Observado:

- `commands/evolve.md:21-23` sí dice que `evolver` se invoca solo si hay evidencia suficiente.
- `commands/evolve.md:41-43` declara como resultado esperado “Manifest creado o
  actualizado” y “Evaluación de cambios previos si aplica”. En una ejecución como
  esta, esa salida esperada entra en tensión con un pedido explícito de
  evaluación-only y con el propio gate de evidencia suficiente.
- `AGENTS.md:67-71` describe el flujo AHE completo y también ubica manifest en
  pasos posteriores al evaluator/debugger, no en el paso evaluator.
- `agents/evaluator.md:41-47` permite crear/actualizar artefactos AHE, pero no
  cerrar la tarea ni implementar fixes.
- `agents/evolver.md:29-34` exige evidencia, predicción y rollback para cambios,
  lo que confirma que el manifest pertenece a evolver, no a evaluator.

Resultado: `fail` por ambigüedad de contrato del comando `/evolve`: el resultado
esperado puede presionar a crear un manifest aun cuando la iteración está en fase
de auditoría/evaluación y el usuario lo prohíbe. No se creó manifest en esta
evaluación.

Reproducibilidad: revisar `commands/evolve.md:19-46` contra el pedido actual y
contra `agents/evaluator.md:31-69` / `agents/evolver.md:23-70`.

### S4: `slash-command-contract-completeness`

Objetivo: comprobar que los contratos de comandos versionados cubren los slash
commands existentes o declaran explícitamente cuáles son auxiliares.

Esperado:

- `docs/ai/harness/commands.md` debería documentar todos los comandos slash
  visibles del repo o marcar una política clara para comandos auxiliares.

Observado:

- `docs/ai/harness/commands.md:3-66` documenta `Mensaje Libre`, `/feature`,
  `/scope`, `/design`, `/evolve` y `/review`.
- Existen además `commands/research.md:1-11`, `commands/spec.md:1-11` y
  `commands/implement.md:1-10`.
- `agents/lead.md:38` y `agents/developer.md:46` recomiendan `/spec` como flujo
  posible ante incertidumbre, pero `/spec` no aparece en `docs/ai/harness/commands.md`.
- `commands/spec.md:7-11` puede producir spec directamente con `specifier`, pero
  `docs/ai/harness/agents.md:22` y `agents/specifier.md:24-47` establecen que
  specifier debe bloquear si falta research/diseño crítico. El contrato existe en
  agente, pero no en el contrato versionado de comando.

Resultado: `fail` por brecha de documentación/orquestación. El sistema tiene
comandos reales no cubiertos por la fuente de verdad declarada
(`docs/ai/harness/README.md:7-13`). Esto reduce trazabilidad de routing y dificulta
evaluar `/research`, `/spec` e `/implement` como escenarios benchmark.

### S5: `design-open-design-tool-contract`

Objetivo: comprobar que la orquestación de `/design` instruye herramientas de
Open Design de forma ejecutable.

Esperado:

- Si un comando ordena usar herramientas de Open Design, sus instrucciones deben
  coincidir con la interfaz disponible o delegar parámetros obligatorios de forma
  inequívoca.

Observado:

- `commands/design.md:16-17` instruye `open_design_health` y
  `open_design_list_agents` “sin argumentos”.
- `commands/design.md:30` prohíbe pasar `baseUrl`.
- En el entorno de esta evaluación, las herramientas Open Design expuestas
  requieren `baseUrl` como parámetro para `open_design_health`,
  `open_design_list_agents`, `open_design_list_design_systems`,
  `open_design_list_skills`, `open_design_create_project` y
  `open_design_run_design`.

Resultado: `fail` por contrato de herramienta potencialmente no ejecutable. Esta
evidencia es estática desde la interfaz de herramientas disponible en esta sesión;
no se ejecutó Open Design para evitar cambiar de alcance y porque la auditoría no
solicitó diseño real.

Impacto potencial: `/design` puede bloquearse antes de producir handoff visual,
lo que a su vez bloquea `/feature` visual antes de `specifier`.

### S6: `transcript-replay-orchestration-prompts`

Objetivo: definir replays reproducibles para validar routing real sin aplicar
cambios.

Estado: `not_run`.

Razón para no ejecutar:

- El workspace está sucio antes de esta evaluación; `git status --short` mostró
  modificaciones/untracked preexistentes, incluyendo `AGENTS.md`,
  `docs/ai/evolution/**`, `docs/ai/harness/`, `docs/console/` y `scripts/`.
- Los replays de OpenCode pueden usar agentes con permisos de edición según el
  comando invocado. Aunque se puede pedir “no edites archivos”, para esta fase
  evaluator se prefirió no iniciar sesiones LLM adicionales que pudieran tocar
  estado o crear ruido en una auditoría sin cambios.

Comando de estado ejecutado:

```bash
git status --short
```

Salida relevante:

```text
 M AGENTS.md
 M docs/ai/evolution/README.md
 M docs/ai/evolution/benchmarks/manual-scenarios.md
 M docs/ai/evolution/evolution_history.md
?? .DS_Store
?? docs/.DS_Store
?? docs/ai/.DS_Store
?? docs/ai/evolution/runs/iteration-003-agent-readable-harness/
?? docs/ai/harness/
?? docs/console/
?? scripts/
```

Replays definidos para una fase posterior segura:

```bash
opencode run --format json --thinking --dir /path/to/opencode \
  "No edites archivos. Para una peticion libre pequena y localizada, indica que agente debe actuar y que barreras minimas aplican."

opencode run --format json --thinking --command feature --dir /path/to/opencode \
  "No edites archivos. Feature tecnica con incertidumbre de API pero sin impacto visual: resume el orden de agentes y que no debe ocurrir antes de spec/developer/reviewer."

opencode run --format json --thinking --command evolve --dir /path/to/opencode \
  "No edites archivos ni crees manifest. Ejecuta solo la fase evaluator de una auditoria AHE y di a quien corresponde el manifest."
```

Evidencia mínima disponible: `static_contract`. Evidencia recomendada antes de
aplicar cambios de routing: `transcript_replay`, según `docs/ai/harness/evidence.md:14-17`.

### S7: `previous-iteration-predictions-risk-tasks`

Objetivo: comprobar si esta auditoría da evidencia sobre cambios previos.

Observado:

- Iteration 001/002: `commands/feature.md:15-19` confirma el predicted fix de
  declarar el flujo base de `/feature` y quitar ambigüedad de orden diseñador/
  researcher. `commands/feature.md:34` confirma que sidecars no se volvieron
  obligatorios para feature normal.
- Iteration 003 `chg-1`: `AGENTS.md:1-74` sigue siendo índice corto y
  `docs/ai/harness/*` contiene contratos versionados.
- Iteration 003 `chg-2`: `node scripts/check-harness.mjs` pasó.
- Iteration 003 `chg-3`: `docs/ai/evolution/benchmarks/manual-scenarios.md:7-27`
  conserva comandos de replay y taxonomía. Esta evaluación no confirma replay
  real nuevo, solo que los prompts reproducibles están definidos.

Resultado: `pass` para evidencia estática de predicted fixes previos. No se
observaron regresiones directas de risk tasks previas, salvo una nueva brecha no
cubierta por el check: contratos de comandos incompletos y contrato `/design`
vs tool schema.

## 4. Comandos o pasos usados

```bash
node scripts/check-harness.mjs
git status --short
node scripts/check-harness.mjs
```

Pasos manuales:

1. Inspección estática de `AGENTS.md`, `docs/ai/harness/*`, comandos y agentes.
2. Comparación de contratos declarados contra comandos reales bajo `commands/`.
3. Comparación de `/design` contra la interfaz de herramientas Open Design
   disponible en esta sesión.
4. Revisión de manifests/evaluaciones previas para predicted fixes y risk tasks.

## 5. Evidencia y rutas relevantes

- Artefacto creado: `docs/ai/evolution/runs/iteration-004-orchestration-audit/evaluation.md`
- Check local inicial: `node scripts/check-harness.mjs` => `Harness check passed.`
- Check local posterior al artefacto evaluator: `node scripts/check-harness.mjs` =>
  `Harness check failed: docs/ai/evolution/runs/iteration-004-orchestration-audit: missing analysis/overview.md`.
- Workspace dirty previo: salida de `git status --short` registrada en S6.
- Contratos principales: `docs/ai/harness/agents.md`, `docs/ai/harness/commands.md`, `docs/ai/harness/evidence.md`, `docs/ai/harness/checks.md`.

## 6. Regresiones observadas

No se observaron regresiones directas de los cambios previos de iteration 001/002/003.
Sí se observaron gaps de orquestación no cubiertos por el validador actual:

1. `scripts/check-harness.mjs` no tolera una iteración AHE en progreso con solo
   `evaluation.md`, aunque `analysis/` corresponde a `debugger`.
2. `/evolve` no expresa claramente un cierre evaluation-only sin manifest.
3. `docs/ai/harness/commands.md` no cubre `/research`, `/spec` ni `/implement`.
4. `/design` parece incompatible con la firma de herramientas Open Design expuesta en esta sesión.

## 7. Limitaciones

- No se ejecutó `opencode run` para evitar riesgo de modificación en workspace
  sucio y porque el usuario pidió no aplicar cambios.
- La comparación de Open Design usa la interfaz de herramientas disponible en
  esta sesión como contrato observable; no se probó contra una sesión real de
  `/design`.
- Esta evaluación no crea `change_manifest.json`; corresponde a `evolver` si
  debugger confirma root cause y evidencia suficiente.
- El repo tiene git, por lo que rollback es posible en principio, pero el
  workspace estaba sucio antes de crear este artefacto; no se debe sobrescribir
  ni revertir cambios de usuario.

## 8. Handoff para debugger/reviewer

Material listo para `debugger`:

- Analizar si S3 es root cause de nivel `command` (`commands/evolve.md`) o
  `workflow` (`docs/ai/harness/commands.md` / AHE docs): contrato de resultado
  presiona manifest aunque el gate diga “solo si evidencia suficiente”.
- Analizar si S1 es root cause de nivel `tool` (`scripts/check-harness.mjs`) o
  `workflow`: el check exige `analysis/overview.md` antes de que el flujo AHE
  llegue a `debugger`.
- Analizar si S4 debe tratarse como `memory`/docs o `command`: comandos auxiliares
  existen pero no tienen contrato en la fuente de verdad versionada.
- Analizar si S5 es root cause de `command`, `skill` o `tool`: `/design` prohíbe
  `baseUrl`, pero las herramientas disponibles lo requieren.

Material listo para `reviewer`:

- Revisar este artefacto contra la petición: no cambia agentes/comandos/skills/tools,
  no crea manifest y preserva cambios existentes.
- Si en fases posteriores se propone cambio, exigir `change_manifest.json` con
  evidence, root cause, predicted fixes, risk tasks y criterio keep/improve/rollback+pivot.
