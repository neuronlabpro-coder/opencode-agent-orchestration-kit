# Pattern: `/evolve` contract does not safely express evaluation-only runs

## Observed evidence

- `commands/evolve.md:21-23` gates evolver: “Invoca a `evolver` solo si hay evidencia suficiente.”
- `commands/evolve.md:39-46` lists expected result including “Manifest creado o actualizado” and “Cambios aplicados, si procede.”
- `AGENTS.md:67-70` places manifest after evaluator/debugger and after sufficient evidence.
- `agents/evolver.md:25-34` says manifest belongs to evolver and requires evidence, prediction, falsifiability and rollback awareness.
- Current user request explicitly says no implementation changes and no `change_manifest.json` in this pass.

## Expected behavior

`/evolve` should support a safe partial AHE pass where evaluator/debugger produce evidence and root causes, then stop if evidence is insufficient or the user explicitly requests evaluation/debugger-only. Manifest creation should be conditional, not an unconditional expected output.

## Root cause

The command file describes the full happy-path AHE closure as the default required outcome, but does not encode an explicit evaluation-only/debugger-only branch. This creates pressure to produce manifest even when the same command gates evolver on sufficient evidence.

## Affected component level

`command` primarily (`commands/evolve.md`), with `workflow` implications for AHE phase boundaries.

## Candidate remediation

- Amend `/evolve` to distinguish full evolution from audit/evaluation-only mode.
- Change result wording to “manifest creado o actualizado solo si procede y fue aprobado”.
- Add stop conditions: user prohibits manifest/implementation, evidence insufficient, or debugger marks no listo.

## Confidence

High for contract ambiguity. The contradictory wording is static and reproducible. Medium for behavioral impact because no transcript replay was run.

## Evidence that would falsify it

- A replay showing `/evolve` reliably respects “no manifest” despite the current result wording.
- A higher-priority policy defining “Resultado esperado” as optional examples rather than command obligations.

## Evidence sufficiency for evolver

Sufficient for a manifest proposing command/workflow clarification. To mark the eventual change keep, run `opencode run --command evolve` with a “no manifest/no edits” prompt and verify it stops after evaluator/debugger handoff.
