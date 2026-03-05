# ADR-0014: Sigil Non-Timeout Stability and Failure Observability Hardening

## Status

Accepted

## Context

Recent harness runs show non-timeout instability modes that are hard to debug:

- node failures are only indirectly visible through `run.failed` or parent
  subcall errors.
- plain subcalls using `sigil.llm.answer.v1` can fail extraction when providers
  return non-JSON raw text despite semantically valid answers.
- REPL compile failures are reduced to flat error strings, limiting adaptive
  next-step correction.

`sigil` needs additive v1 hardening that improves terminal visibility and
non-timeout failure recovery without changing CLI/run-config surfaces.

## Decision

1. Node-failure terminalization is explicit via new runtime event type
   `node.failed`.
2. Plain-subcall extraction fallback is schema-specific:
   - applies only to `sigil.llm.answer.v1`
   - if structured extraction fails and non-empty raw text is available,
     normalize to `{"answer":"<trimmed raw text>"}`.
3. Structured compile diagnostics are persisted in action artifacts and fed into
   subsequent step `previous_action_feedback.error_detail`.
4. `node.action.executed` payload shape remains unchanged; diagnostics are
   artifact/feedback only.

## Decision Details

- Canonical node terminal event set becomes:
  - success: `node.completed`
  - failure: `node.failed`
- Lifecycle enforces terminal-node exclusivity:
  - exactly one of `node.completed` or `node.failed` per started node in normal
    harness execution.
- Failure ordering rules:
  - child failure path: child `node.failed` MUST be emitted before parent
    `node.subcall.executed` failed record.
  - root failure path: root `node.failed` MUST be emitted before `run.failed`.
- Fallback observability:
  - adapter emits extraction-mode details in `raw_metadata` when raw-text
    fallback is used.
- Compile diagnostics contract:
  - `error_detail.stage` literal `compile`
  - include message and optional line/column/symbol/source_line when parseable
    from compile error text.

## Consequences

- Node terminal outcomes become directly observable and queryable from event
  logs.
- Plain-subcall stability improves for borderline provider output formatting
  without weakening strict schema validation globally.
- Compile-stage corrective feedback to the model becomes more actionable without
  inflating canonical runtime event payloads.

## Deferred

- Runtime event payload expansion for rich diagnostics beyond
  `node.failed` terminal metadata.
- Raw-text fallback for schemas other than `sigil.llm.answer.v1`.
- Structured runtime (non-compile) diagnostic parsing.
