# ADR-0013: Sigil Structured Output Verifiability and Prompt-Schema Sync

## Status

Accepted

## Context

`sigil.rlm.response.v1` currently enforces decision branching and minimal
payload fields, but it does not require explicit step intent metadata or final
evidence references. This weakens auditability for recursive retrieval
workloads and makes final-answer traceability harder to enforce.

The current system prompts also embed a static schema block that can drift from
the runtime schema registry over time.

## Decision

1. `sigil.rlm.response.v1` is extended in-place (pre-release hard cutover):
   - `decision=continue` requires:
     - `continuation.repl_code` (non-empty)
     - `continuation.intent` (non-empty)
     - `continuation.expected_observation` (non-empty)
   - `decision=final` requires:
     - `final.answer` (non-empty)
     - `final.evidence` (array, minItems=1)
     - optional `final.confidence` enum: `low|medium|high`
2. Evidence entries are strict objects with:
   - `ref` (required)
   - optional `chunk_id`
   - optional `span_start`
   - optional `span_end` (must be `>= span_start` when both are present)
3. Harness step-input context metadata includes mandatory `context_ref`
   pointing to a persisted node context artifact.
4. Final evidence refs are strict-resolve only:
   - unresolved refs are output-validation failures
   - run terminates as failed with typed output-validation metadata
5. Prompt/schema parity is runtime-derived:
   - the system prompt schema block is generated from
     `schema.SigilRLMResponseV1SchemaID` at runtime
   - prompt schema text and strict validation schema share one source of truth
6. Provider prompts move to in-code source strings in
   `sigil/internal/harness/system_prompts.go`.
   External prompt template files are removed.

## Consequences

- Final answers become evidence-bound and machine-auditable.
- First-step final answers remain possible because `context_ref` is always
  available as a resolvable evidence anchor.
- Prompt/schema drift risk is materially reduced by runtime schema rendering.
- This is a hard cutover and requires updating fixtures/tests that still emit
  legacy v1 payload shape.

## Deferred

- Cross-run evidence references.
- Confidence calibration beyond fixed enum values.
- Rich citation typing beyond v1 `evidence[]` object.
