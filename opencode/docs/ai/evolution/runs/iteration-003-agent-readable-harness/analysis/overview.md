# Debugger Analysis: iteration-003-agent-readable-harness

## Evidencia Analizada

- `AGENTS.md`
- `docs/ai/harness/README.md`
- `docs/ai/harness/agents.md`
- `docs/ai/harness/commands.md`
- `docs/ai/harness/evidence.md`
- `docs/ai/harness/checks.md`
- `docs/ai/evolution/benchmarks/manual-scenarios.md`
- `scripts/check-harness.mjs`
- `docs/ai/evolution/runs/iteration-003-agent-readable-harness/evaluation.md`

## Estado

Listo para `keep` en esta iteración.

## Root Cause

La mejora parte de una limitación estructural observada en el harness: las reglas
globales, contratos, evidencia y rutinas de validación estaban repartidas entre
`AGENTS.md`, agentes, comandos y `docs/ai/evolution/`, pero no existía un mapa
versionado específico para que los agentes encontraran esos contratos sin cargar
un manual largo.

## Fix Candidato

Mover detalle estable a `docs/ai/harness/`, mantener `AGENTS.md` como índice,
añadir un check local sin dependencias y subir el benchmark desde oráculo manual
hacia replays con eventos JSON.

## Hallazgos

- El nuevo validador confirmó los contratos después de dos ajustes: ESM en `.mjs`
  y parsing top-level de frontmatter.
- `AGENTS.md` conserva las reglas críticas: modo directo a `developer`, flujos
  slash explícitos, sidecars opcionales y AHE con manifest/evaluación.
- El replay real confirmó el contrato de mensaje libre pequeño: `developer` es
  el agente esperado.
- No hay evidencia de que `evaluator`, `debugger` o `evolver` se hayan convertido
  en fases obligatorias del flujo normal.

## Riesgos

- Documentación duplicada si `AGENTS.md` vuelve a crecer.
- Check demasiado rígido si intenta validar comportamiento en vez de contratos.
- Replay insuficiente si se interpreta como prueba completa de runtime.

## Recomendación

Mantener los cambios. La siguiente iteración útil debería ejecutar replays de
`/feature` y `/scope` para comprobar barreras reales con eventos JSON.
