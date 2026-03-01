# ADR-0006: Sigil Run/Node Identity Model

## Status

Accepted

## Date

2026-03-01

## Context

`sigil` recursive execution requires explicit identity semantics for:

- Run-level traceability.
- Recursive tree structure (root and child nodes).
- Durable event correlation across runtime and future projections.

Without a formal identity model, recursive lineage and cross-event
correlation become ambiguous and difficult to validate.

## Decision

`sigil` MUST use a run/node identity model with UUIDv7 identifiers and explicit
root/child node invariants.

## Decision Details

- `run_id`, `node_id`, and `event_id` MUST be UUIDv7 strings.
- Every run MUST have exactly one root node where:
  - `depth=0`
  - `parent_node_id=null`
- Child nodes MUST reference a parent node in the same run.
- `node_id` uniqueness is global; event correlation still uses `run_id` plus
  `node_id`.
- Tool/code-exec activity MUST NOT create node entities in v1; it is represented
  as node-scoped events.
- Retry attempts SHOULD be represented via explicit attempt metadata on
  node-scoped events.

## Alternatives Considered

- Run-local node IDs only: rejected because cross-run exports and merged views
  require composite-key handling everywhere.
- Path-encoded node IDs (for example `0.1.2`): rejected due to complexity for
  retries and potential tree reshaping.
- Modeling tool/code execution as separate node entities in v1: rejected to keep
  core identity graph focused on recursive units.

## Consequences

- Recursive lineage is explicit and deterministic.
- Event and projection contracts can depend on stable global node identifiers.
- v1 identity scope remains simple: recursion nodes as entities,
  tool/code-exec as node-scoped activity.

## Migration/Adoption Notes

- Existing and future runtime components should emit and consume UUIDv7 identity
  values consistently.
- Identity invariants should be validated during event append and replay.
- Any move to additional node kinds in future versions requires a superseding
  ADR or explicit PRD contract update.

## Related Documents

- [Sigil ADR Index](README.md)
- [ADR-0005 Event-Sourced Run Architecture](ADR-0005-sigil-event-sourced-run-architecture.md)
- [PRD-0006 Run Lifecycle State Machine Specification](../PRD/PRD-0006-sigil-run-lifecycle-state-machine-specification.md)
- [PRD-0007 Run Event Contract Specification](../PRD/PRD-0007-sigil-run-event-contract-specification.md)
- [Project Spec Rules](../../../../.agents/rules/SPECS.md)
