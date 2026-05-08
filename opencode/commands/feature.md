---
description: Orchestrate a complete feature through discovery, spec, implementation, and review.
agent: lead
---


Objective:

$ARGUMENTS

Run the feature flow with explicit barriers.

## Mandatory flow

1. Analyze the objective only enough to classify scope, uncertainty, and
   handoffs; do not turn this phase into technical discovery or broad
   repository/config inspection.
2. Apply the base flow: `lead -> designer if applicable -> researcher -> specifier -> developer -> reviewer`.
3. Decide whether design and/or research are needed with minimal routing
   context. Lead's pre-handoff inspection is limited to obvious signals needed
   to choose the next agent.
4. If the request states or reveals UX, brand, layout, interaction, or visual
   acceptance-criteria uncertainty, invoke `designer` before the first
   substantive discovery on those topics and wait for handoff.
5. If the request states or reveals technical/product/API/library/risk/
   architecture uncertainty, invoke `researcher` before the first substantive
   discovery, broad implementation/config reading, or technical conclusion by
   `lead`, then wait for output.
6. The early handoff to `researcher`/`designer` must include the objective,
   detected uncertainty, constraints, and expected evidence; it must not
   pre-resolve the question that motivated delegation.
7. Do not make `researcher` universal: if the feature is simple, clear, and has
   no relevant uncertainty, keep fast routing toward spec/implementation as
   appropriate.
8. Parallelize designer and researcher only if their results are independent.
9. Synthesize required design/research before invoking `specifier`.
10. Invoke `developer` only for sufficiently specified tasks.
11. Invoke `reviewer` after implementation.
12. If reviewer requires changes, route through lead back to developer, then review again.
13. Close with changes, validation, and risks.

AHE sidecars are optional and must not be inserted unless evidence is needed or this request changes the harness itself.
