# Pattern: `/design` Open Design instructions contradict available tool schemas

## Observed evidence

- `commands/design.md:16-17` instructs `open_design_health` and `open_design_list_agents` “sin argumentos”.
- `commands/design.md:30` says “No pases `baseUrl`”.
- The Open Design tools available in this session require `baseUrl` for health, listing, create-project, and run-design calls.
- `docs/ai/harness/commands.md:38-47` says `/design` is `designer -> open design`, so this command depends on the tool contract being executable.

## Expected behavior

If `/design` mandates Open Design tools, its instructions must match the actual callable tool interface or state exactly how the required `baseUrl` is obtained/delegated.

## Root cause

The `/design` command embeds tool-call instructions that are stale or environment-assumptive relative to the current exposed Open Design tool schema. It forbids the same required parameter the tools demand.

## Affected component level

`command` primarily (`commands/design.md`). `tool` is affected only if the exposed schema is wrong; current evidence points to command/schema mismatch, not tool implementation failure.

## Candidate remediation

- Update `/design` to load the `open-design` skill and follow its documented source for `baseUrl`, or pass an explicit approved `baseUrl` when calling tools.
- Remove the blanket “No pases `baseUrl`” rule unless the tool interface changes to make it optional.
- Add a validation scenario: health + list agents/design systems/skills against the configured base URL without creating design artifacts.

## Confidence

High for static schema mismatch in this session. Medium for runtime impact because no `/design` smoke was executed.

## Evidence that would falsify it

- A successful `/design` replay showing the skill injects a wrapper/default `baseUrl` despite command text forbidding it.
- Tool schema changes making `baseUrl` optional or automatically resolved.

## Evidence sufficiency for evolver

Sufficient for a manifest proposing command/tool-contract correction. Not sufficient to claim runtime success until an Open Design health/list smoke runs with the corrected contract.
