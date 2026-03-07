# ADR-0011: Sigil Harness Model-Input Boundary and Message Contract

## Status

Accepted

## Context

`sigil` currently executes the RLM harness with a persistent node-local Go REPL
and strict structured outputs. However, model-step inference input is assembled
by replaying broad context content each step.

For long-context RLM behavior, this weakens the architecture goal: keep full
context in the execution environment (REPL), while model turns operate on a
bounded control envelope.

## Decision

For v1 Priority 1, `sigil` adopts a hard-cutover model-input boundary:

- Full raw run `context` is REPL-local and MUST NOT be replayed into model-step
  inference input each step.
- Inference request contracts move from prompt/context string fields to ordered
  role-based messages.
- Harness model-step input is a deterministic bounded user envelope containing:
  - root query
  - step index
  - context metadata
  - optional previous-action feedback summary
- Previous-action feedback summary previews are capped at:
  - `stdout_preview`: 2048 bytes
  - `stderr_preview`: 2048 bytes
  and include truncation metadata.
- `node.turn.user` output artifacts persist compact metadata plus references and
  MUST NOT embed full raw context content.
- Migration is hard cutover (no compatibility shim).

## Consequences

- RLM behavior aligns with context-boundary intent for large contexts.
- Token pressure per step is reduced by preventing full-context replay.
- Inference and gateway contracts become message-centric, enabling cleaner
  provider alignment.
- Acceptance and unit coverage must be updated for message-array request shape
  and compact user-turn artifacts.

## Non-Goals

- Changing `sigil.rlm.response.v1` structured response schema.
- Introducing configurable feedback-preview caps in v1.
- Altering REPL session lifecycle in `PRD-0430` or guardrail contracts in
  `PRD-0500`.
