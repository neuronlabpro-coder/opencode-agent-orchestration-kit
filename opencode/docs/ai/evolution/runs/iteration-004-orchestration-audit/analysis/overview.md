# Debugger Analysis: iteration-004-orchestration-audit

## 1. Evidencia analizada

- `evaluation.md` de esta iteración, especialmente S1, S3, S4, S5, S6 y S7.
- `scripts/check-harness.mjs` líneas 149-164, que valida runs AHE.
- Contratos del harness: `AGENTS.md`, `docs/ai/harness/{README,agents,commands,evidence,checks}.md`.
- Comandos reales: `commands/{evolve,design,research,spec,implement}.md` y listado de `commands/*.md`.
- Agentes AHE: `agents/{evaluator,debugger,evolver}.md`.
- Iteración previa: `iteration-003-agent-readable-harness/change_manifest.json` y `change_evaluation.json`.
- Interfaz observable de herramientas Open Design expuesta en esta sesión: las tools requieren `baseUrl`.

Limitaciones: no se ejecutó `opencode run` ni smoke real de Open Design; el workspace ya estaba sucio según `evaluation.md`. Hay git, pero esta pasada no revisa ni revierte diff de usuario.

## 2. Estado

Estado global: **listo para evolver con alcance acotado**, salvo que cualquier cambio propuesto afecte routing real; esos cambios deben incluir `transcript_replay` posterior antes de declararse keep.

Los cuatro fallos confirmados son independientes y no deben unificarse en un único manifest/change.

## 3. Patrones de fallo

1. **Validador AHE no modela estados intermedios legítimos** (`tool`): `check-harness` falla cuando existe `evaluation.md` sin `analysis/overview.md`, aunque esa es la salida esperada antes de debugger.
   - Detalle: `analysis/detail/ahe-run-lifecycle-validator.md`.
2. **Contrato de `/evolve` mezcla flujo completo con modo evaluation-only** (`command`/`workflow`): el gate “evolver solo si hay evidencia suficiente” entra en tensión con “Manifest creado o actualizado”.
   - Detalle: `analysis/detail/evolve-evaluation-only-contract.md`.
3. **Fuente de verdad de comandos incompleta frente a comandos reales** (`memory`/`command`): `commands.md` omite `/research`, `/spec` e `/implement` o una política de comandos auxiliares.
   - Detalle: `analysis/detail/slash-command-contract-coverage.md`.
4. **Contrato `/design` incompatible con firma de Open Design tools expuestas** (`command`/`tool`): instruye llamar tools sin argumentos y prohíbe `baseUrl`, pero los schemas requieren `baseUrl`.
   - Detalle: `analysis/detail/open-design-tool-contract.md`.

## 4. Root causes

- **RC1:** `scripts/check-harness.mjs` trata todo directorio bajo `docs/ai/evolution/runs/` como run completo y no distingue fases AHE en progreso.
- **RC2:** `commands/evolve.md` define un resultado final normativo para el flujo completo pero no representa explícitamente el cierre seguro cuando el usuario pide solo evaluación/debugging o prohíbe manifest.
- **RC3:** La documentación versionada de comandos se convirtió en fuente de verdad declarada, pero su cobertura no se sincronizó con todos los archivos `commands/*.md` ni declara qué comandos son auxiliares/subtask.
- **RC4:** `commands/design.md` conserva un contrato antiguo o asumido de Open Design que contradice la interfaz actual disponible en el harness.

## 5. Fixes candidatos por nivel de componente

- **Tool:** ajustar `check-harness` para aceptar estados AHE intermedios explícitos, sin debilitar validación de runs cerrados ni manifests existentes.
- **Command/workflow:** aclarar `/evolve` con un modo evaluation-only/debugger-only y salida “manifest solo si procede”.
- **Memory/command:** actualizar `docs/ai/harness/commands.md` para cubrir `/research`, `/spec`, `/implement` o declarar una política verificable de comandos auxiliares; considerar check de cobertura.
- **Command/tool:** corregir el contrato de `/design` para obtener o delegar `baseUrl` de forma ejecutable y consistente con la skill `open-design`/schemas disponibles.

## 6. Riesgos y regresiones

- Relajar demasiado el validador podría ocultar runs AHE incompletos abandonados.
- Hacer `/evolve` demasiado flexible podría omitir manifest cuando sí hubo cambio de harness.
- Documentar comandos auxiliares sin barreras puede permitir que `/spec` salte research/diseño crítico.
- Corregir `/design` sin smoke podría seguir fallando si el `baseUrl` esperado proviene de skill/config no visible en el contrato estático.

## 7. Atribución de cambios previos

- Iteration 003 `chg-1` (`memory`): **improve**. La dirección de crear `docs/ai/harness` como fuente de verdad se confirma, pero S4 muestra cobertura incompleta de comandos reales, una variante del riesgo `harness-docs-duplicate-command-files`.
- Iteration 003 `chg-2` (`tool`): **improve**. Predicted fixes principales siguen útiles, pero S1 confirma el risk task `validator-blocks-valid-future-run-shape`: el validador bloquea un estado intermedio AHE válido.
- Iteration 003 `chg-3` (`workflow`): **keep con limitación**. La taxonomía de evidencia evita sobredeclarar routing sin replay; S6 está correctamente `not_run` por workspace sucio. No hay regresión confirmada de sus risk tasks.

No se escribe `change_evaluation.json` porque el lead no lo pidió en esta pasada y el encargo prohíbe manifest/cambios de implementación.

## 8. Handoff para evolver/reviewer

Evolver puede proponer un manifest con hasta cuatro cambios independientes, uno por patrón. Alcance recomendado:

1. `tool`: lifecycle explícito en `check-harness` para runs AHE en progreso.
2. `command`: clarificación de `/evolve` para evaluation-only y manifest-gated.
3. `memory/command`: cobertura o política de comandos auxiliares en `docs/ai/harness/commands.md`, con posible check de drift.
4. `command/tool`: contrato ejecutable para Open Design `baseUrl` en `/design`.

Reviewer debe exigir que el manifest mantenga estos patrones separados y que cualquier cambio que prometa routing real incluya replay posterior. Para `/design`, exigir smoke o al menos health/list tools con `baseUrl` tras el cambio antes de marcar keep.
