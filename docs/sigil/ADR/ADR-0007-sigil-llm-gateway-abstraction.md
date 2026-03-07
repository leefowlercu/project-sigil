# ADR-0007: Sigil LLM Gateway Abstraction

## Status

Accepted

## Date

2026-03-01

## Context

`sigil` requires a long-lived architecture for LLM inference that supports:

- Decoupling harness runtime behavior from any specific external gateway service.
- Decoupling gateway integration from underlying provider/model specifics.
- Future expansion to multiple gateway implementations without changing harness
  core inference orchestration.

Without a formal abstraction boundary, introducing additional gateway services
later would force broad runtime refactors.

## Decision

`sigil` MUST implement LLM inference using a `Gateway interface + registry +
adapter` architecture pattern.

## Decision Details

- Harness inference core MUST depend on a gateway interface only.
- Gateway resolution MUST be performed through a registry keyed by
  `llm.gateway`.
- Each concrete gateway integration MUST be implemented as an adapter.
- Adapter layer MUST own transport/request/response mapping details for that
  gateway.
- OpenRouter-specific transport/request/response details MUST NOT leak into
  harness-core inference contracts.

## Alternatives Considered

- Direct single-gateway integration in harness core: rejected because it couples
  core logic to gateway-specific API semantics.
- Factory-only switching without registry contract: rejected because it weakens
  extension/discoverability guarantees as gateway count grows.
- Strategy-chain orchestration for v1: rejected because it adds unnecessary
  complexity before multi-gateway runtime behavior is required.

## Consequences

- Future gateway additions become additive adapter work rather than harness-core
  rewrites.
- Gateway selection semantics become explicit and testable.
- Core inference contracts remain stable across gateway implementations.

## Migration/Adoption Notes

- Existing inference integrations should move behind adapter boundaries as
  touched.
- Gateway-specific payload quirks should be normalized in adapters, not in
  runtime orchestration logic.
- This ADR defines architecture only; behavior-level contracts are specified in
  PRDs.

## Related Documents

- [Sigil ADR Index](README.md)
- [ADR-0008 OpenRouter Responses Gateway](ADR-0008-sigil-openrouter-responses-gateway.md)
- [PRD-0120 Run Config Core Specification](../PRD/PRD-0120-sigil-run-config-core-specification.md)
- [PRD-0300 Inference Gateway Specification](../PRD/PRD-0300-sigil-inference-gateway-specification.md)
- [Project Spec Rules](../../../../.agents/rules/SPECS.md)
