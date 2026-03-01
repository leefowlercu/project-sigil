# ADR-0004: Use slog for Structured JSON File Logging

## Status

Accepted

## Date

2026-03-01

## Context

`sigil` needs a single, standard logging implementation approach that supports:

- Structured logs suitable for machine parsing.
- Low-friction integration with the Go runtime and ecosystem.
- Consistent logging behavior across command and runtime flows.

The project also needs clear separation between architecture-level logging
implementation choices and behavior-level logging output contracts.

## Decision

`sigil` logging implementation MUST use Go standard library `log/slog`.

## Decision Details

- Structured logging output MUST use `slog` JSON handler behavior.
- Logging output target behavior is specified as a product behavior contract in
  `PRD-0005-sigil-application-logging-specification.md`.
- This ADR defines the implementation framework requirement and does not replace
  PRD acceptance behavior.

## Alternatives Considered

- Third-party logging libraries (`zap`, `zerolog`, `logrus`): rejected to avoid
  additional dependency and policy divergence from standard-library baseline.
- Legacy unstructured logging (`log` package): rejected because it does not
  provide equivalent structured JSON logging semantics.

## Consequences

- Runtime logging integrations in `sigil` align on `log/slog`.
- Structured JSON logging semantics are standardized by default.
- Migrating to non-`slog` logging in the future requires a superseding ADR.

## Migration/Adoption Notes

- Existing or future logging code should standardize on `slog` as touched.
- Behavior-level log destination and acceptance expectations remain governed by
  PRD contracts.

## Related Documents

- [Sigil ADR Index](README.md)
- [PRD-0005 Application Logging Specification](../PRD/PRD-0005-sigil-application-logging-specification.md)
- [Project Spec Rules](../../../../.agents/rules/SPECS.md)
