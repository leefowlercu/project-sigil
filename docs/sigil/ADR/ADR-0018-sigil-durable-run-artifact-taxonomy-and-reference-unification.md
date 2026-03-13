# ADR-0018: Sigil Durable Run Artifact Taxonomy and Reference Unification

## Status

Accepted

## Date

2026-03-12

## Context

`sigil` already uses append-only per-run events as its source of truth, but its
durable non-event storage vocabulary is split across:

- `artifacts/` with `run-artifact://...` refs for action execution records
- `outputs/` with `run-output://...` refs for context, turns, final answers, and
  accounting outputs

That split creates avoidable complexity:

- operators and future rich clients have to reason about three durable nouns:
  `events`, `artifacts`, and `outputs`
- `node.action.executed.output_ref` already points to a `run-artifact://...`
  action artifact, which makes the field name misleading
- app-server query and ref-resolution surfaces become harder to name cleanly

Sigil needs one durable taxonomy that keeps the event-sourced source of truth
intact while making every non-event persisted record feel like one coherent
artifact model.

## Decision

Sigil durable run data adopts this taxonomy:

- `events`
  - append-only canonical runtime history in `events.jsonl`
- `artifacts`
  - every other durable run-scoped persisted record

## Decision Details

- `events.jsonl` remains the only source of truth for runtime state.
- All durable non-event records MUST live under the run `artifacts/` tree.
- The `outputs/` durable storage family is retired from the steady-state
  contract.
- Canonical non-event refs MUST use one artifact namespace.
  - v1 target prefix is `run-artifact://...`
- Semantic role-oriented field names MAY remain when they describe meaning
  rather than storage family.
  - examples: `context_ref`, `content_ref`, `final_answer_ref`,
    `accounting_ref`
- Misleading output-shaped names SHOULD be renamed to artifact-accurate names.
  - `node.action.executed.output_ref` -> `action_ref`
  - `previous_action_feedback.output_ref` -> `action_ref`
  - `read_action_output(output_ref)` -> `read_action_artifact(action_ref)`
- Durable artifacts remain immutable file-per-artifact JSON records in v1.
- Sigil does NOT introduce per-kind artifact JSONL streams in v1.

## Alternatives Considered

- Keep `outputs/` and `artifacts/` as separate durable categories: rejected
  because the split adds vocabulary and query complexity without clear runtime
  value.
- Use a second append-only JSONL store for artifacts: rejected because event
  ordering already lives in `events.jsonl`, while artifact access optimizes for
  exact ref resolution.
- Keep existing `output_ref` naming even after ref unification: rejected because
  it would preserve the current terminology mismatch in the new taxonomy.

## Consequences

### Positive

- Sigil runtime vocabulary becomes easier to explain: a durable thing is either
  an event or an artifact.
- Rich-client and app-server read surfaces can standardize on `artifact/read`.
- Ref resolution no longer needs to special-case multiple non-event namespaces.
- Future run inspection and UI contracts become easier to reason about.

### Negative

- This change touches multiple contracts across events, projections, prompts,
  evidence handling, and acceptance scenarios.
- Pre-release runtime payload and helper names will change in a breaking way.
- Prompt and model-input wording must be updated carefully to avoid drift.

## Migration/Adoption Notes

- This is a specs-first change: update ADRs, PRDs, matrix rows, acceptance
  titles, glossary terms, and app-server plans before the Sigil submodule
  implementation lands.
- Because Sigil is still pre-release, the preferred path is a clean cutover
  rather than long-lived compatibility aliases.

## Related Documents

- [Sigil ADR Index](README.md)
- [ADR-0005 Event-Sourced Run Architecture](ADR-0005-sigil-event-sourced-run-architecture.md)
- [ADR-0009 Go REPL Engine and Runtime Boundary](ADR-0009-sigil-go-repl-engine-and-runtime-boundary.md)
- [ADR-0016 Accounting Ledger and Provenance Contract](ADR-0016-sigil-accounting-ledger-and-provenance-contract.md)
- [PRD-0210 Run Event Contract Specification](../PRD/PRD-0210-sigil-run-event-contract-specification.md)
- [PRD-0310 Structured Response Schema and Evidence Specification](../PRD/PRD-0310-sigil-structured-response-schema-and-evidence-specification.md)
- [PRD-0420 Model Input Boundary and Step Feedback Specification](../PRD/PRD-0420-sigil-model-input-boundary-and-step-feedback-specification.md)
- [PRD-0430 Go REPL Runtime Specification](../PRD/PRD-0430-sigil-go-repl-runtime-specification.md)
- [PRD-0510 Accounting Ledger Specification](../PRD/PRD-0510-sigil-accounting-ledger-specification.md)
- [Project Spec Rules](../../../../.agents/rules/SPECS.md)
