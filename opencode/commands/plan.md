---
description: Create an implementation-ready plan without applying changes: research, spec, and plan review.
agent: lead
---


Objective:

$ARGUMENTS

Run a deterministic plan-only flow.

Contract: `lead -> researcher -> specifier -> reviewer`.

## Mandatory flow

1. Ground the user objective with assumptions, questions, scope, non-scope, and verifiable success criteria.
2. Always invoke `researcher` first; wait for the full result before continuing.
3. Synthesize research findings: verified facts, inferences, risks, recommended decisions, and open questions.
4. Invoke `specifier` only after research has been consolidated.
5. Ask `specifier` for an implementation-ready plan, acceptance criteria, validation, and ordered tasks.
6. Invoke `reviewer` to review the plan/spec against the objective, research, assumptions, risks, and acceptance criteria.
7. If `reviewer` finds no significant issues, deliver the final plan.
8. If `reviewer` finds significant issues, allow exactly 1 correction pass.
9. After the second review, deliver the plan with explicit risks or declare the state blocked.

## Dependency rules

- Do not invoke developer.
- Do not invoke `designer`; if the work needs design, mark that dependency in the plan or recommend `/design` or `/feature`.
- Do not implement code or modify application files.
- Do not invoke `specifier` while `researcher` has pending work.
- Do not invoke `reviewer` before a reviewable plan/spec exists.
- Do not use `evaluator`, `debugger`, or `evolver`; this command plans, it does not evolve or validate the harness.
- Do not parallelize phases; this flow is sequential by design.
- If there is a blocking question, stop the flow and state it instead of inventing a decision.

## Single correction pass

If `reviewer` returns `requires changes`:

- If evidence, context, or research is missing, send a bounded task back to `researcher` and then to `specifier`.
- If research is sound but the plan is incomplete or inconsistent, send a bounded task only to `specifier`.
- If there is an unresolvable contradiction or blocking product decision, stop and deliver a blocked state.
- After that correction, invoke a second and final `reviewer` pass.
- Do not run additional cycles even if observations remain; deliver risks, pending items, and the next human decision.

## Expected output

Deliver:

1. Research summary.
2. Decisions made.
3. Assumptions and open questions.
4. Implementation-ready plan.
5. Acceptance criteria.
6. Validation plan.
7. Reviewer result.
8. Risks and pending items.
