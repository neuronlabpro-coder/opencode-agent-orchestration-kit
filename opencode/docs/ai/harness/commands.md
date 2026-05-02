# Command Contracts

## Free-form Message

Contract: use `developer` by default.

Criteria:

- If the change is small, clear, and low risk, implement with minimum validation.
- If there is real uncertainty or medium/large scope, ask or recommend
  `/feature`, `/scope`, `/design`, or `/spec`.

## `/feature`

Contract: `lead -> designer if applicable -> researcher -> specifier -> developer -> reviewer`.

Criteria:

- `designer` acts before `specifier` if UX, brand, layout, interaction, or visual
  acceptance criteria matter.
- `researcher` acts before `specifier` if technical uncertainty, APIs,
  libraries, risks, or architecture matter.
- `designer` and `researcher` may run in parallel only if their results are
  independent.
- `developer` does not act without acceptance criteria.
- `reviewer` does not act without a reviewable diff.

## `/scope`

Contract: `scoper -> researcher -> scoper synthesis -> specifier`.

Criteria:

- No design, implementation, or diff review.
- `debugger` enters only for traces, results, or concrete previous evidence.
- Output is scoped specs, atomic tasks, and validation.

## `/design`

Contract: `designer -> open design`.

Criteria:

- Read `PRODUCT.md` and `DESIGN.md` when they exist.
- Use `impeccable` only when product/design context is missing.
- Use `open-design` for editable project or visual generation as requested.

## `/evolve`

Full contract for real harness changes: `evaluator -> debugger -> evolver -> lead approval -> developer -> evaluator -> debugger -> reviewer`.

Allowed no-apply branches:

- `evaluation-only`: stops after `evaluator` when the user requested audit or
  evaluation only and did not authorize analysis, manifest, or implementation.
- `debugger-only` / `no-apply` / `no-manifest`: stops after `debugger` with root
  causes, falsification criteria, and handoff; it does not invoke `evolver` or
  `developer`.

Criteria:

- No changes without concrete evidence.
- Every applied change needs a valid `change_manifest.json` and approval to
  apply.
- Manifest, developer, and implementation are conditional on sufficient
  evidence, user scope, and approval; they are not mandatory outputs of
  evaluation-only or debugger-only branches.
- The next measurement needs `change_evaluation.json` when a manifest/change was
  applied or previous changes are being evaluated.
- Do not promise automatic rollback without git.

## `/review`

Contract: `reviewer`.

Criteria:

- Review `git diff`, active spec, and available evidence.
- If the diff changes the harness, also review AHE manifests and evaluations.
