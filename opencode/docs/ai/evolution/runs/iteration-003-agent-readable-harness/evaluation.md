# Evaluation: iteration-003-agent-readable-harness

## Objetivo Evaluado

Aplicar la mejora AHE 003: hacer el harness más legible para agentes, mover
contratos detallados fuera de `AGENTS.md`, añadir checks mecánicos locales y
preparar benchmarks para `transcript_replay`.

## Escenarios

| Escenario | Evidencia | Resultado |
| --- | --- | --- |
| `AGENTS.md` funciona como índice corto | `static_contract` | pass |
| Docs de harness versionan agentes, comandos, evidencia y checks | `static_contract` | pass |
| Validador local comprueba contratos mínimos | `static_contract` | pass |
| Benchmarks documentan `opencode run --format json --thinking` | `static_contract` | pass |
| Al menos un replay real de OpenCode se ejecuta y queda registrado | `transcript_replay` | pass |

## Comandos Planificados

```bash
node scripts/check-harness.mjs
opencode run --format json --thinking --dir /path/to/opencode "Responde sin editar archivos: indica que agente deberia manejar una peticion libre pequena y por que."
```

## Resultados

### Static Contract

Comando:

```bash
node scripts/check-harness.mjs
```

Resultado: `pass`.

Notas:

- La primera ejecución detectó un bug real del nuevo script: `.mjs` corre como
  ESM y no acepta `require`.
- La segunda ejecución detectó que el parser era demasiado estricto con YAML
  anidado.
- Tras ajustar el script a `import` y validar solo claves top-level obligatorias,
  el check pasó.

### Transcript Replay

Comando:

```bash
opencode run --format json --thinking --dir /path/to/opencode "Responde sin editar archivos: indica que agente deberia manejar una peticion libre pequena y por que."
```

Resultado: `pass`.

Evidencia observada:

- Session ID: `ses_21a7fa327ffewVJqpvfeHFm9ST`.
- El replay emitió eventos JSON.
- La respuesta final indicó que una petición libre pequeña debería manejarla
  `developer`.
- La justificación citó el modo directo para peticiones sin `/comando`,
  pequeñas, claras y de bajo riesgo.

## Atribución

- `chg-1`: `keep` para la evidencia estática actual. `AGENTS.md` queda como
  índice corto y `docs/ai/harness/` contiene contratos versionados.
- `chg-2`: `keep`. El validador pasó y además detectó dos errores reales durante
  su propio endurecimiento.
- `chg-3`: `keep` para esta iteración. El benchmark ya documenta replay y el
  replay real confirma el contrato de mensaje libre pequeño.

## Limitaciones

- Esta iteración no cambia modelos, proveedores, credenciales ni permisos.
- Los replays son evidencia de routing/contrato, no de una feature real de app.
- Solo se ejecutó un replay real; los demás escenarios quedan preparados para
  futuras iteraciones.
