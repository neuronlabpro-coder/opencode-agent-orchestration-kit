# Post-apply Evaluation: iteration-004 orchestration audit

## 1. Objetivo evaluado

Validar la aplicación post-implementation del alcance aprobado: solo `chg-1-ahe-run-lifecycle-validator` y `chg-2-evolve-evaluation-only-contract`. Confirmar que `chg-3-slash-command-contract-coverage` y `chg-4-design-open-design-baseurl-contract` permanecen diferidos/no aplicados, y que no se tocaron archivos de proveedores, modelos o credenciales.

## 2. Escenarios ejecutados

1. Check local completo del harness.
2. Fixture positiva temporal: run evaluator-only con `evaluation.md`, sin `analysis/overview.md` y sin `change_manifest.json`.
3. Fixture/control negativa temporal: run con `change_manifest.json`, sin `analysis/overview.md` y sin `change_evaluation.json`.
4. Parse JSON de `change_manifest.json` y verificación de estados approved/applied/deferred.
5. Static contract check de `/evolve` y `docs/ai/harness/commands.md`.
6. Revisión de nombres en diff/untracked contra scope permitido y exclusión de provider/model/credential files.

## 3. Resultados por escenario

| Escenario | Resultado | Evidencia |
| --- | --- | --- |
| `node scripts/check-harness.mjs` | pass | Salida: `Harness check passed.` |
| Fixture evaluator-only | pass | Temp: `/var/folders/6n/gwypgcsn171gfjh1268kj3380000gn/T/opencode/ahe-fixture-1777677951825`; salida: `Harness check passed.` |
| Fixture/control manifest sin debugger/cierre | pass | Temp: `/var/folders/6n/gwypgcsn171gfjh1268kj3380000gn/T/opencode/ahe-control-1777678040034`; salida esperada fallida con `missing analysis/overview.md` y `manifest exists without change_evaluation.json`; `EXIT_STATUS=1`. |
| Manifest JSON y estados | pass | JSON parse OK; `chg-1`/`chg-2`: `implemented_applied`; `chg-3`/`chg-4`: `deferred_not_approved`; `approved_changes` solo chg-1/chg-2; `deferred_changes` chg-3/chg-4. |
| Static contract `/evolve` | pass | Checks verdaderos: evaluation-only, debugger-only/no-apply/no-manifest, stop sin manifest/evolver, full AHE preservado, manifest requerido para cambio aplicado. |
| Diff scope y archivos sensibles | pass | Archivos cambiados/untracked dentro de allowlist; sin matches provider/model/credential/secret/token/key. |

## 4. Comandos o pasos usados

```bash
node scripts/check-harness.mjs
```

Fixture positiva creada bajo temp aprobado mediante script Node; luego ejecutada con:

```bash
node scripts/check-harness.mjs
```

Resultado fixture positiva: `Harness check passed.`

Fixture/control negativa creada bajo temp aprobado mediante script Node; luego ejecutada esperando fallo:

```bash
node scripts/check-harness.mjs
```

Resultado fixture/control negativa:

```text
Harness check failed:
- docs/ai/evolution/runs/manifest-without-analysis: missing analysis/overview.md
- docs/ai/evolution/runs/manifest-without-analysis: manifest exists without change_evaluation.json
EXIT_STATUS=1
```

Validaciones estáticas adicionales ejecutadas con scripts Node locales para:

- parsear `docs/ai/evolution/runs/iteration-004-orchestration-audit/change_manifest.json`;
- verificar tokens/contratos en `commands/evolve.md` y `docs/ai/harness/commands.md`;
- listar `git status --short`, `git diff --name-only`, `git diff --cached --name-only` y `git ls-files --others --exclude-standard` para scope de archivos.

## 5. Evidencia y rutas relevantes

- `scripts/check-harness.mjs`: acepta runs evaluator-only sin manifest y conserva fallo para manifest sin análisis/cierre.
- `docs/ai/harness/checks.md`: documenta lifecycle AHE intermedio y regla estricta cuando existe manifest.
- `commands/evolve.md`: documenta stop conditions evaluation-only, debugger-only, no-apply y no-manifest, preservando flujo completo para cambios reales.
- `docs/ai/harness/commands.md`: separa contrato completo de `/evolve` de ramas permitidas sin aplicación.
- `docs/ai/evolution/runs/iteration-004-orchestration-audit/change_manifest.json`: estados post-apply aprobados/deferidos verificados.

Archivos en diff/untracked al momento de evaluación:

```text
commands/evolve.md
docs/ai/evolution/runs/iteration-004-orchestration-audit/analysis/pre_apply_approval.md
docs/ai/evolution/runs/iteration-004-orchestration-audit/change_manifest.json
docs/ai/evolution/runs/iteration-004-orchestration-audit/post_apply_evaluation.md
docs/ai/evolution/runs/iteration-004-orchestration-audit/pre_apply_evaluation.md
docs/ai/harness/checks.md
docs/ai/harness/commands.md
scripts/check-harness.mjs
```

## 6. Predicted fixes confirmados / no confirmados

### Confirmados para chg-1

- `check-harness-allows-evaluator-only-ahe-run-state`: confirmado por fixture positiva.
- `check-harness-still-fails-manifest-without-debugger-analysis`: confirmado por fixture/control negativa.
- `ahe-step-1-and-step-2-validation-no-longer-blocked-by-premature-complete-run-shape`: confirmado parcialmente por check local + fixture evaluator-only. No se ejecutó transcript replay completo de `/evolve`.

### Confirmados para chg-2

- `evolve-audit-only-stops-after-evaluator-or-debugger-when-requested`: confirmado por static contract; no se ejecutó transcript replay.
- `evolve-manifest-created-only-when-evolver-has-sufficient-evidence-and-user-scope-allows-it`: confirmado por static contract.
- `evolve-no-apply-requests-do-not-pressure-developer-implementation`: confirmado por static contract.

### No confirmados

- Transcript replay real con `opencode run --format json --thinking` para ramas evaluation-only/debugger-only/full-flow: not_run por alcance de esta validación post-apply y para evitar crear ejecución larga/no solicitada más allá de checks indicados.

## 7. Riesgos observados / no observados

### No observados

- `validator-accepts-abandoned-incomplete-runs-too-broadly`: no observado en fixture/control con manifest; el validador sigue fallando cuando hay manifest sin análisis/cierre.
- `manifest-shape-errors-hidden-by-lifecycle-exception`: no observado para runs con manifest; el control entró en ruta estricta.
- `check-harness-lifecycle-rules-become-hard-to-understand`: no observado en docs; `checks.md` explica la regla en dos bullets.
- `evolve-skips-required-manifest-after-actual-harness-change`: no observado estáticamente; `commands.md` conserva manifest obligatorio para cambio aplicado.
- `ahe-full-flow-less-obvious-to-users`: mitigado parcialmente; `commands/evolve.md` mantiene pasos completos y `commands.md` mantiene contrato completo.
- Cambios de proveedores/modelos/credenciales: no observados por diff filename scan.

### Riesgos residuales

- La confirmación de comportamiento de agentes para `/evolve` sigue siendo static_contract, no transcript_replay.
- `docs/ai/harness/commands.md` fue tocado por chg-2, que comparte archivo con chg-3/chg-4; la inspección confirmó que no aparecen `commands/design.md` ni cobertura nueva de `/research`/`/spec`/`/implement`, pero una revisión de diff semántica por reviewer sigue recomendada.

## 8. Limitaciones

- No se ejecutó `opencode run --format json --thinking`; por tanto, no hay evidencia transcript_replay de cumplimiento operacional del lead ante prompts evaluation-only/debugger-only/full-flow.
- Las fixtures temporales se crearon fuera del repo bajo `/var/folders/6n/gwypgcsn171gfjh1268kj3380000gn/T/opencode/` y no se conservaron como artefacto versionado.
- Esta evaluación no modifica implementación ni cierra la tarea; entrega evidencia para debugger/reviewer/lead.

## 9. Handoff para debugger/reviewer

El debugger puede producir `change_evaluation.json` con decisión candidata `keep` para chg-1 y chg-2, sujeta a declarar que la evidencia de chg-2 es estática y no transcript_replay. Si debugger exige prueba operacional de `/evolve`, la decisión debería ser `improve` con tarea de replay, no `rollback+pivot`, porque no hay regresión observada en checks/fixtures ni archivos fuera de scope. `rollback+pivot` no está justificado por esta evidencia.
