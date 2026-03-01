# ADR-0003: Use Godog for BDD Acceptance Testing

## Status

Accepted

## Date

2026-03-01

## Context

`sigil` needs an acceptance-testing approach that captures externally observable
behavior in executable scenarios and remains distinct from implementation-level
unit testing.

The testing strategy must:

- Keep behavior expectations readable and reviewable by product and engineering
  stakeholders.
- Preserve a clear separation between unit and acceptance testing layers.
- Align acceptance execution with Gherkin scenarios as behavioral source of
  truth.

## Decision

`sigil` acceptance testing MUST use `github.com/cucumber/godog` for Gherkin-based
BDD scenarios.

Unit testing MUST continue to use Go standard library `testing`.

## Decision Details

- Acceptance scenarios MUST be authored as Gherkin and executed through Godog.
- Acceptance tests MUST assert externally observable behavior (black-box),
  not internal implementation details.
- Unit tests and acceptance tests MUST remain separate layers with distinct
  intent.
- Unit tests SHOULD remain fast, deterministic, and implementation-focused.
- Acceptance tests SHOULD map each scenario to observable outcomes.
- Behavior changes SHOULD update acceptance scenarios first, then implementation.
- PRD acceptance criteria remain the source-of-truth layer for product behavior;
  this ADR defines the `sigil` implementation framework for executing those
  behavioral scenarios.

## Alternatives Considered

- Plain `go test` only for all behavior levels: rejected because it weakens
  shared, behavior-first scenario readability.
- Alternative BDD frameworks: rejected to avoid divergence from selected
  ecosystem standards and existing policy direction.
- Custom acceptance harness format: rejected due to maintenance burden and lower
  interoperability.

## Consequences

- `sigil` acceptance behavior contracts become consistently executable through
  Godog.
- Test-layer boundaries (unit vs acceptance) become explicit and enforceable.
- Deviating from Godog for acceptance testing requires a superseding ADR.

## Migration/Adoption Notes

- New acceptance coverage should be added as Godog scenarios.
- Existing acceptance coverage in other formats should be migrated to Godog as
  touched or expanded.
- This ADR defines architecture-level testing framework direction only; it does
  not itself add or alter acceptance scenario content.

## Related Documents

- [Sigil ADR Index](README.md)
- [Sigil PRD Matrix](../PRD/MATRIX.md)
- [Project Spec Rules](../../../../.agents/rules/SPECS.md)
- [Project Workflow Rules](../../../../.agents/rules/WORKFLOW.md)
