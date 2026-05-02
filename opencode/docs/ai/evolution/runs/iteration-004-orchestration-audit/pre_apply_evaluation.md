# Pre-apply evaluation: approved chg-1/chg-2 only

## 1. Objetivo evaluado

Validar, antes de implementación, si las propuestas aprobadas por el usuario en
`change_manifest.json` tienen evidencia suficiente y son consistentes con el
historial AHE:

- `chg-1-ahe-run-lifecycle-validator`
- `chg-2-evolve-evaluation-only-contract`

Alcance excluido explícitamente: no evaluar para aplicación, implementar ni
recomendar `chg-3-slash-command-contract-coverage` ni
`chg-4-design-open-design-baseurl-contract`.

## 2. Escenarios ejecutados

| ID | Escenario | Tipo | Resultado |
| --- | --- | --- | --- |
| PA1 | Consistencia histórica con iteration 003 y decisiones keep/improve previas | `static_contract` | pass |
| PA2 | Manifest readiness para `chg-1` y `chg-2` | `static_contract` | pass |
| PA3 | Baseline `node scripts/check-harness.mjs` en repo actual | comando local | pass |
| PA4 | Fixture temporal evaluator-only para lifecycle AHE | temp smoke estático | fail esperado pre-apply |
| PA5 | Fixture temporal manifest-sin-analysis para control negativo | temp smoke estático | pass como control negativo |
| PA6 | Contrato actual de `/evolve` para evaluation-only/debugger-only | `static_contract` | fail esperado pre-apply |

## 3. Resultados por escenario

### PA1: consistencia histórica

Resultado: `pass`.

Evidencia inspeccionada:

- `docs/ai/evolution/evolution_history.md:19-36` marca iteration 003 como `keep`
  e identifica `docs/ai/harness/` como fuente de verdad y
  `scripts/check-harness.mjs` como validador local.
- `iteration-003-agent-readable-harness/change_manifest.json` registra:
  - `chg-1`: mover contratos a `docs/ai/harness/`.
  - `chg-2`: añadir validador local sin dependencias.
  - `chg-3`: documentar replay y taxonomía de evidencia.
- `iteration-003-agent-readable-harness/change_evaluation.json` mantiene `keep`
  para los tres cambios y registra como risk task de `chg-2`:
  `validator-blocks-valid-future-run-shape`.
- `iteration-004-orchestration-audit/analysis/overview.md:53-57` atribuye:
  - iteration 003 `chg-1`: `improve` por brecha de cobertura de comandos.
  - iteration 003 `chg-2`: `improve` porque se confirmó el riesgo
    `validator-blocks-valid-future-run-shape`.
  - iteration 003 `chg-3`: `keep con limitación`.

Interpretación:

- `chg-1-ahe-run-lifecycle-validator` responde directamente a evidencia nueva
  sobre un riesgo ya previsto del validador de iteration 003; no contradice el
  `keep`, lo refina.
- `chg-2-evolve-evaluation-only-contract` aclara un contrato de comando sin
  cambiar la fuente de verdad de iteration 003; no contradice decisiones previas.
- No se necesita aplicar `chg-3` ni `chg-4` para que `chg-1`/`chg-2` sean
  consistentes históricamente.

### PA2: readiness de manifest para cambios aprobados

Resultado: `pass`.

Comando:

```bash
node -e "const fs=require('fs'); const p='docs/ai/evolution/runs/iteration-004-orchestration-audit/change_manifest.json'; const m=JSON.parse(fs.readFileSync(p,'utf8')); const wanted=['chg-1-ahe-run-lifecycle-validator','chg-2-evolve-evaluation-only-contract']; for (const id of wanted) { const c=m.changes.find(x=>x.id===id); if(!c) throw new Error('missing '+id); const fields=['evidence','root_cause','predicted_fixes','risk_tasks','validation_plan']; const missing=fields.filter(f=>c[f]===undefined || (Array.isArray(c[f]) && c[f].length===0) || c[f]===''); if(missing.length) throw new Error(id+' missing '+missing.join(',')); console.log(id+': fields ok; evidence='+c.evidence.length+' predicted='+c.predicted_fixes.length+' risks='+c.risk_tasks.length+' validation='+c.validation_plan.length); }"
```

Salida:

```text
chg-1-ahe-run-lifecycle-validator: fields ok; evidence=6 predicted=3 risks=3 validation=3
chg-2-evolve-evaluation-only-contract: fields ok; evidence=6 predicted=3 risks=3 validation=3
```

Además, la inspección estática confirmó que cada cambio aprobado tiene
`root_cause`, `predicted_fixes`, `risk_tasks`, `validation_plan`,
`why_this_component` y `rollback_plan`.

### PA3: baseline harness actual

Resultado: `pass`.

Comando:

```bash
node scripts/check-harness.mjs
```

Salida:

```text
Harness check passed.
```

Interpretación: el repo actual está en una forma completa que satisface el
validador existente antes de aplicar cambios. Esto da baseline para que developer
pueda medir regresiones después.

### PA4: fixture temporal evaluator-only

Resultado: `fail esperado pre-apply`.

Fixture creado solo bajo:

`/var/folders/6n/gwypgcsn171gfjh1268kj3380000gn/T/opencode/ahe-preapply-fixture`

Comando resumido:

```bash
node -e "...crea fixture mínimo con docs/ai/evolution/runs/iteration-temp-evaluator-only/evaluation.md...; process.chdir(base); require('child_process').execFileSync('node',['/path/to/opencode/scripts/check-harness.mjs'],{stdio:'inherit'});"
```

Salida relevante:

```text
Harness check failed:
- docs/ai/evolution/runs/iteration-temp-evaluator-only: missing analysis/overview.md
```

Interpretación: reproduce de forma aislada la falla que `chg-1` pretende
resolver. El resultado es un fail del sistema actual, no una regresión de este
pase.

### PA5: fixture temporal manifest-sin-analysis

Resultado: `pass como control negativo`.

Fixture creado solo bajo:

`/var/folders/6n/gwypgcsn171gfjh1268kj3380000gn/T/opencode/ahe-preapply-manifest-no-analysis`

Comando resumido:

```bash
node -e "...crea fixture mínimo con evaluation.md y change_manifest.json sin analysis/overview.md ni change_evaluation.json...; process.chdir(base); require('child_process').execFileSync('node',['/path/to/opencode/scripts/check-harness.mjs'],{stdio:'inherit'});"
```

Salida relevante:

```text
Harness check failed:
- docs/ai/evolution/runs/iteration-temp-manifest: missing analysis/overview.md
- docs/ai/evolution/runs/iteration-temp-manifest: manifest exists without change_evaluation.json
```

Interpretación: confirma el control negativo requerido por `chg-1`: una
implementación futura no debe aceptar de forma amplia un manifest sin análisis ni
evaluación de cambio. Post-apply, este caso debería seguir fallando de manera
explícita.

### PA6: contrato actual de `/evolve`

Resultado: `fail esperado pre-apply`.

Comando:

```bash
node -e "const fs=require('fs'); const cmd=fs.readFileSync('commands/evolve.md','utf8'); const map=fs.readFileSync('docs/ai/harness/commands.md','utf8'); const hasMandatoryManifest=cmd.includes('Manifest creado o actualizado'); const hasExplicitEvalOnly=/evaluation-only|evaluaci[oó]n-only|solo evaluaci[oó]n|debugger-only/i.test(cmd+'\n'+map); console.log('commands/evolve mandatory manifest wording: '+hasMandatoryManifest); console.log('explicit evaluation-only/debugger-only branch wording: '+hasExplicitEvalOnly); if(!hasMandatoryManifest) process.exitCode=2;"
```

Salida:

```text
commands/evolve mandatory manifest wording: true
explicit evaluation-only/debugger-only branch wording: false
```

Interpretación: reproduce estáticamente la ambigüedad que `chg-2` pretende
resolver. La evidencia es suficiente para una aclaración de contrato; el replay
de comportamiento queda como validación post-apply antes de marcar `keep`.

## 4. Comandos o pasos usados

```bash
git status --short --branch
node scripts/check-harness.mjs
node -e "...validación de campos de chg-1/chg-2..."
node -e "...fixture temporal evaluator-only..."
node -e "...fixture temporal manifest-sin-analysis..."
node -e "...inspección estática de wording /evolve..."
```

También se inspeccionaron manualmente:

- `docs/ai/evolution/evolution_history.md`
- `docs/ai/evolution/runs/iteration-003-agent-readable-harness/{evaluation.md,analysis/overview.md,change_manifest.json,change_evaluation.json}`
- `docs/ai/evolution/runs/iteration-004-orchestration-audit/{evaluation.md,analysis/overview.md,change_manifest.json,change_evaluation.json,post_manifest_evaluation.md}`
- `docs/ai/evolution/runs/iteration-004-orchestration-audit/analysis/detail/{ahe-run-lifecycle-validator.md,evolve-evaluation-only-contract.md}`
- `scripts/check-harness.mjs`
- `commands/evolve.md`
- `docs/ai/harness/{checks.md,commands.md}`

## 5. Evidencia y rutas relevantes

- Artefacto actual: `docs/ai/evolution/runs/iteration-004-orchestration-audit/pre_apply_evaluation.md`
- Manifest: `docs/ai/evolution/runs/iteration-004-orchestration-audit/change_manifest.json`
- Evidencia original: `docs/ai/evolution/runs/iteration-004-orchestration-audit/evaluation.md`
- Root causes: `docs/ai/evolution/runs/iteration-004-orchestration-audit/analysis/overview.md`
- Detalle `chg-1`: `docs/ai/evolution/runs/iteration-004-orchestration-audit/analysis/detail/ahe-run-lifecycle-validator.md`
- Detalle `chg-2`: `docs/ai/evolution/runs/iteration-004-orchestration-audit/analysis/detail/evolve-evaluation-only-contract.md`
- Temp fixtures: `/var/folders/6n/gwypgcsn171gfjh1268kj3380000gn/T/opencode/ahe-preapply-*`

## 6. Regresiones observadas

No se observaron regresiones nuevas durante esta evaluación. No se aplicaron
cambios de implementación. Los dos failures son esperados en estado pre-apply y
corresponden a los comportamientos que `chg-1`/`chg-2` intentan corregir.

## 7. Limitaciones

- No se ejecutó `opencode run --command evolve`; queda como validación post-apply
  para confirmar comportamiento real de `chg-2` antes de marcar `keep`.
- Las fixtures temporales son minimalistas y ejercitan el validador local, no una
  sesión completa del harness.
- Este pase no modifica scripts, comandos, agentes, providers, modelos ni
  credenciales.
- `chg-3` y `chg-4` permanecen fuera de alcance aunque existan en el manifest.

## 8. Handoff para debugger/reviewer/developer

- Evidencia suficiente para proceder con developer únicamente en el alcance
  aprobado: `chg-1-ahe-run-lifecycle-validator` y
  `chg-2-evolve-evaluation-only-contract`.
- Developer debe evitar `chg-3` y `chg-4` y limitar archivos a los declarados por
  `chg-1`/`chg-2`: `scripts/check-harness.mjs`, `docs/ai/harness/checks.md`,
  `commands/evolve.md`, `docs/ai/harness/commands.md`.
- Validación post-apply mínima recomendada:
  1. `node scripts/check-harness.mjs` en repo actual.
  2. Fixture evaluator-only debe pasar o reportar estado in-progress explícito.
  3. Fixture manifest-sin-analysis debe seguir fallando.
  4. Inspección estática de `/evolve` debe mostrar rama evaluation-only/debugger-only.
  5. Si el lead lo autoriza, replay `opencode run --command evolve` no-mutante.
