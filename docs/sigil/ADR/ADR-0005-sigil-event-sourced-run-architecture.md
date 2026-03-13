# ADR-0005: Sigil Event-Sourced Run Architecture

## Status

Accepted

## Date

2026-03-01

## Context

`sigil` needs a durable and extensible runtime foundation for recursive run
execution that can support:

- Durable run artifacts and replayability.
- Stable contracts for future app-server read/control APIs.
- Deterministic recovery and integrity validation.

The current CLI and run specifications define command surface and configuration
behavior, but do not yet define the authoritative runtime persistence model for
run execution.

## Decision

`sigil` runtime state MUST use an event-sourced architecture where append-only
per-run events are the source of truth.

## Decision Details

- Runtime source of truth MUST be append-only per-run events.
- Projections/read models MUST be derived from events and are not source of
  truth.
- Durable non-event run records MUST be treated as artifacts rather than as a
  second source of truth.
- v1 durable store default MUST be JSONL in a per-run directory under
  `./.sigil/runs`.
- Event ordering MUST rely on per-run contiguous `seq` values.
- Query/app-server contracts are deferred to future PRDs and are not specified
  by this ADR.

## Alternatives Considered

- Mutable current-state records as primary source of truth: rejected because it
  weakens replay, auditability, and deterministic recovery.
- Projection-first storage without durable event history: rejected because it
  prevents authoritative reconstruction after corruption or migration.
- Database-only storage as mandatory baseline: rejected for v1 due to higher
  operational and implementation complexity versus JSONL baseline needs.

## Consequences

- Runtime history becomes replayable and auditable by construction.
- Non-event durable records remain queryable without weakening event authority.
- Future app-server and UI read models can be implemented as projections over a
  stable write-path contract.
- Sequence integrity becomes explicit and testable.
- Evolving event schemas requires version-aware readers/writers.

## Migration/Adoption Notes

- Existing and future run persistence behavior should align to append-only
  per-run events as touched.
- Initial v1 durable format is JSONL; future storage backends may be added if
  they preserve event-sourced source-of-truth semantics.
- CLI UX contracts in `PRD-0110` and `PRD-0130` remain unchanged in this
  iteration.

## Related Documents

- [Sigil ADR Index](README.md)
- [ADR-0006 Run/Node Identity Model](ADR-0006-sigil-run-node-identity-model.md)
- [PRD-0200 Run Lifecycle State Machine Specification](../PRD/PRD-0200-sigil-run-lifecycle-state-machine-specification.md)
- [PRD-0210 Run Event Contract Specification](../PRD/PRD-0210-sigil-run-event-contract-specification.md)
- [Project Spec Rules](../../../../.agents/rules/SPECS.md)
