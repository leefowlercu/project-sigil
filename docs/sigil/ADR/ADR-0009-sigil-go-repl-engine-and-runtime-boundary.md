# ADR-0009: Sigil Go REPL Engine and Runtime Boundary

## Status

Accepted

## Date

2026-03-02

## Context

`sigil` now defines harness-level recursive control behavior in `PRD-0400`, but
it does not yet lock the architecture boundary for how Go REPL execution is
hosted, isolated by node, and integrated with event contracts.

The project requires a stable v1 architecture that supports:

- Deterministic node-local REPL state across continue steps.
- Recursive subcalls via `rlm_query` without coupling harness logic to one REPL
  engine implementation.
- Durable action-artifact references in runtime events.
- A clear migration path to stricter sandboxing in a later version.

## Decision

`sigil` v1 REPL runtime MUST use an embedded in-process Go interpreter
(yaegi-backed) with one persistent session per node and an internal session
abstraction boundary.

## Decision Details

- REPL engine for v1 is embedded and in-process.
- Exactly one REPL session exists per node.
- Node REPL sessions are persistent across continue steps and are not recreated
  per step.
- Harness runtime owns REPL session lifecycle:
  - create lazily on first continue action for node
  - close on node completion
  - close all remaining sessions when run reaches terminal state
- REPL runtime boundary MUST be defined via internal interfaces (`Session`,
  `SessionFactory`) so the engine can be replaced without changing harness
  contracts.
- REPL action execution artifacts MUST be persisted under run-local storage and
  referenced via `node.action.executed.action_ref`.
- Fixed v1 runtime timeout budgets are:
  - REPL action timeout: `180s`
  - Recursive REPL subcall timeout: `300s`
- Recursive subcalls (`rlm_query`, `rlm_query_batched`) MUST execute with a
  run-scoped context source and independent timeout budget rather than
  inheriting parent action elapsed deadline or ancestor recursive subcall
  deadline depletion.
- Import policy in v1 MUST be allowlist-based only.
- Non-fatal REPL execution errors MUST be fed back into model context and MUST
  NOT automatically fail the run.

## Alternatives Considered

- Out-of-process REPL runtime in v1: rejected to avoid first-release complexity
  and process orchestration overhead while contracts stabilize.
- Ephemeral REPL per action: rejected because it breaks node-local state
  continuity and increases runtime overhead.
- Direct engine coupling in harness: rejected because it prevents controlled
  engine swaps and complicates future sandbox migration.

## Consequences

- V1 achieves deterministic stateful step execution without process startup
  overhead per action.
- Harness and REPL implementation remain decoupled via interface boundary.
- Future migration to out-of-process sandboxing remains additive rather than a
  harness contract rewrite.
- Security hardening beyond allowlists and fixed limits remains deferred.

## Migration/Adoption Notes

- Behavior-level REPL runtime contracts and limits are defined in
  `PRD-0430`.
- Event payload invariants for action artifacts remain defined by `PRD-0210`.
- Harness-level recursive behavior remains defined by `PRD-0400`.

## Related Documents

- [Sigil ADR Index](README.md)
- [ADR-0007 LLM Gateway Abstraction](ADR-0007-sigil-llm-gateway-abstraction.md)
- [ADR-0008 OpenRouter Responses Gateway](ADR-0008-sigil-openrouter-responses-gateway.md)
- [PRD-0210 Run Event Contract Specification](../PRD/PRD-0210-sigil-run-event-contract-specification.md)
- [PRD-0400 Harness Control Loop Specification](../PRD/PRD-0400-sigil-harness-control-loop-specification.md)
- [PRD-0430 Go REPL Runtime Specification](../PRD/PRD-0430-sigil-go-repl-runtime-specification.md)
- [Project Spec Rules](../../../../.agents/rules/SPECS.md)
