# PRD-0310: Sigil Structured Response Schema and Evidence Specification

## Status

Draft

## Context

`sigil` needs one normative owner for structured response schemas and final
answer evidence semantics.

This PRD owns:

- central schema registry usage
- `sigil.rlm.response.v1`
- `sigil.llm.answer.v1`
- final-answer evidence reference rules
- final-answer artifact persistence

Inference transport behavior is defined in `PRD-0300`. Prompt composition and
runtime schema rendering are defined in `PRD-0320`. Model-input context metadata
and `previous_action_feedback` shape are defined in `PRD-0420`.

## Goals

- Define the canonical structured response schemas used by harness inference.
- Define evidence requirements for `decision=final`.
- Define plain-subcall answer schema ownership.
- Define validation and persistence rules for final-answer evidence.

## Non-Goals

- Defining gateway retry or healing behavior.
- Defining event payload schemas.
- Defining cross-run evidence resolution.
- Defining backward-compatibility shims for legacy payload shapes.

## Central Schema Registry Contract

- Inference schema resolution MUST use the central registry only.
- Inline ad-hoc schema definitions are out of contract.
- Required schema IDs are:
  - `sigil.rlm.response.v1`
  - `sigil.llm.answer.v1`
- Unresolved `schema_id` MUST fail request construction with a typed schema-lookup error.

## Structured Response Contract: `sigil.rlm.response.v1`

`sigil.rlm.response.v1` defines the required shape for `validated_payload`.

Top-level fields:

- `decision` (required string enum): `continue`, `final`
- `continuation` (optional object):
  - `repl_code` (required, non-empty when `decision=continue`)
  - `intent` (required, non-empty when `decision=continue`)
  - `expected_observation` (required, non-empty when `decision=continue`)
- `final` (optional object):
  - `answer` (required, non-empty when `decision=final`)
  - `evidence` (required array with at least one item when `decision=final`)
  - `confidence` (optional enum: `low`, `medium`, `high`)

Evidence item fields:

- `ref` (required string, non-empty)
- `chunk_id` (optional string)
- `span_start` (optional integer `>= 0`)
- `span_end` (optional integer `>= span_start` when both are present)

Schema invariants:

- `decision=continue` MUST include `continuation` and MUST NOT include `final`.
- `decision=final` MUST include `final` and MUST NOT include `continuation`.
- Unknown top-level or nested fields MUST fail strict schema validation.

## Structured Response Contract: `sigil.llm.answer.v1`

`sigil.llm.answer.v1` defines strict plain-subcall output shape.

Top-level fields:

- `answer` (required string, non-empty)

Schema invariants:

- Payload MUST include `answer`.
- Unknown top-level fields MUST fail strict schema validation.

## Evidence Resolution Contract

- `final.evidence[].ref` MUST resolve before node completion.
- Accepted canonical schemes in this release are:
  - `run-output://...`
  - `run-artifact://...`
- When final evidence cites a prior action artifact surfaced via `previous_action_feedback.output_ref`, the cited ref MUST match the provided `output_ref` byte-for-byte.
- If exact reuse of `previous_action_feedback.output_ref` is not possible, final evidence MUST cite `context_ref` instead of synthesizing a new `run-artifact://...` ref.
- Unresolvable refs MUST fail finalization with typed output-validation behavior.

## Final Output Persistence Contract

- Final-answer artifacts MUST persist:
  - `answer`
  - `evidence[]`
  - optional `confidence`
- `run.completed.final_answer_ref` continues to point at the persisted final-answer artifact.

## Compatibility Contract

- Continue-step one-action-per-step behavior is unchanged.
- Recursive and non-recursive execution profiles are unchanged.
- Inference request `schema_id` remains `sigil.rlm.response.v1` after schema enrichment in this release.

## Deferred Contracts

- Cross-run evidence resolution
- user-configurable evidence validation modes
- schema migration or downgrade behavior across versions
- backward-compatibility shims for legacy response shapes

## Acceptance Scenarios

### Scenario SCN-0000: Resolves schema_id sigil.rlm.response.v1 from central registry for inference requests

Given an inference request with `schema_id=sigil.rlm.response.v1`  
When schema resolution runs  
Then the schema is resolved through the central registry.

### Scenario SCN-0001: Rejects inference request when schema_id is not found in central registry

Given an inference request references an unknown `schema_id`  
When schema resolution runs  
Then request construction fails with typed schema-lookup behavior.

### Scenario SCN-0002: Requires decision discriminator values continue or final in sigil.rlm.response.v1

Given a payload validated against `sigil.rlm.response.v1`  
When strict schema validation runs  
Then `decision` must be `continue` or `final`.

### Scenario SCN-0003: Requires continuation intent expected_observation and repl_code in continue branch

Given an inference payload with `decision=continue`  
When strict schema validation runs  
Then `continuation.repl_code`, `continuation.intent`, and
`continuation.expected_observation` are required and non-empty.

### Scenario SCN-0004: Requires continuation repl_code and forbids final branch when decision is continue

Given an inference payload with `decision=continue`  
When strict schema validation runs  
Then `continuation` is required and `final` is forbidden.

### Scenario SCN-0005: Requires final answer evidence and optional confidence enum in final branch

Given an inference payload with `decision=final`  
When strict schema validation runs  
Then `final.answer` and `final.evidence` are required and
`final.confidence` is optional with enum `low|medium|high`.

### Scenario SCN-0006: Requires final branch and forbids continuation branch when decision is final

Given an inference payload with `decision=final`  
When strict schema validation runs  
Then `final` is required and `continuation` is forbidden.

### Scenario SCN-0007: Restricts final confidence to enum low medium or high when present

Given an inference payload with `decision=final` and `final.confidence` present  
When strict schema validation runs  
Then confidence is limited to `low`, `medium`, or `high`.

### Scenario SCN-0008: Rejects unknown fields and malformed evidence entries under strict schema

Given inference payload contains unknown fields or invalid evidence span data  
When strict schema validation runs  
Then the payload is rejected with typed output-validation behavior.

### Scenario SCN-0009: Resolves schema_id sigil.llm.answer.v1 from central registry for plain subcall inference requests

Given an inference request with `schema_id=sigil.llm.answer.v1`  
When schema resolution runs  
Then the schema is resolved through the central registry.

### Scenario SCN-0010: Requires non-empty answer field in sigil.llm.answer.v1 payloads

Given a payload validated against `sigil.llm.answer.v1`  
When strict schema validation runs  
Then `answer` is required and non-empty.

### Scenario SCN-0011: Exposes canonical and resolvable evidence references through context_ref and previous_action_feedback.output_ref

Given a step context with `context_ref` and a prior action with
`previous_action_feedback.output_ref`  
When evidence references are evaluated  
Then those refs are canonical and resolvable.

### Scenario SCN-0012: Validates final evidence references against run-local persisted artifacts before node completion

Given a final payload cites one or more evidence refs  
When node finalization runs  
Then each evidence ref is validated against run-local persisted artifacts before completion.

### Scenario SCN-0013: Fails run with typed output-validation metadata when any final evidence reference cannot be resolved

Given a final payload cites an unresolvable evidence ref  
When node finalization runs  
Then finalization fails with typed output-validation metadata.

### Scenario SCN-0014: Accepts final evidence references for canonical run-output and run-artifact schemes

Given a final payload cites canonical `run-output://...` or `run-artifact://...` refs  
When evidence validation runs  
Then those schemes are accepted.

### Scenario SCN-0015: Persists enriched final-answer artifact with answer evidence and optional confidence

Given a final payload passes validation  
When final-answer persistence runs  
Then the persisted artifact contains answer, evidence, and optional confidence.

### Scenario SCN-0016: Requires byte-for-byte previous_action_feedback.output_ref reuse with context_ref fallback for final evidence citations

Given final evidence cites a prior action artifact  
When evidence refs are constructed or validated  
Then `previous_action_feedback.output_ref` is reused byte-for-byte, or
`context_ref` is used as fallback instead of synthesizing a new artifact ref.

### Scenario SCN-0017: Maintains inference schema_id sigil.rlm.response.v1 after schema extension

Given the structured response contract is enriched within this release  
When inference requests are built  
Then the request `schema_id` remains `sigil.rlm.response.v1`.
