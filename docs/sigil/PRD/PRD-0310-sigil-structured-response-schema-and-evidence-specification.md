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
- Define action-based final-answer evidence requirements.
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

- `decision` (required string enum): `continue`
- `continuation` (required object):
  - `repl_code` (required, non-empty)
  - `intent` (required, non-empty)
  - `expected_observation` (required, non-empty)

Schema invariants:

- `decision=continue` MUST include `continuation`.
- `final` MUST be absent.
- `decision=final` is out of contract.
- Unknown top-level or nested fields MUST fail strict schema validation.

## Structured Response Contract: `sigil.llm.answer.v1`

`sigil.llm.answer.v1` defines strict plain-subcall output shape.

Top-level fields:

- `answer` (required string, non-empty)

Schema invariants:

- Payload MUST include `answer`.
- Unknown top-level fields MUST fail strict schema validation.

## Evidence Resolution Contract

- Final-answer evidence MUST come from the completed action artifact whose
  output contained the accepted `FINAL_ANSWER_START` / `FINAL_ANSWER_END`
  block.
- Accepted canonical scheme in this release is:
  - `run-artifact://...`
- When REPL code cites or recovers prior action artifacts via
  `previous_action_feedback.action_ref`, the cited ref MUST match the provided
  `action_ref` byte-for-byte.
- Unresolvable refs MUST fail finalization with typed output-validation behavior.

## Final Output Persistence Contract

- Final-answer artifacts MUST persist:
  - `answer`
  - `evidence[]`
- `run.completed.final_answer_ref` continues to point at the persisted final-answer artifact.

## Compatibility Contract

- Every completed step is a continue step with exactly one action.
- Recursive and non-recursive execution profiles are unchanged.
- Inference request `schema_id` remains `sigil.rlm.response.v1` after the
  action-only schema change in this release.

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

### Scenario SCN-0002: Requires action-only continue discriminator in sigil.rlm.response.v1

Given a payload validated against `sigil.rlm.response.v1`  
When strict schema validation runs  
Then `decision` must be `continue`.

### Scenario SCN-0003: Requires continuation intent expected_observation and repl_code in continue branch

Given an inference payload with `decision=continue`  
When strict schema validation runs  
Then `continuation.repl_code`, `continuation.intent`, and
`continuation.expected_observation` are required and non-empty.

### Scenario SCN-0004: Requires continuation repl_code and forbids final branch when decision is continue

Given an inference payload with `decision=continue`  
When strict schema validation runs  
Then `continuation` is required and `final` is forbidden.

### Scenario SCN-0005: Rejects direct final decision payloads in sigil.rlm.response.v1

Given an inference payload with `decision=final`  
When strict schema validation runs  
Then the payload is rejected with typed output-validation behavior.

### Scenario SCN-0006: Persists final-answer artifact from marked action output

Given a completed action emits a canonical marked final-answer block  
When node finalization runs  
Then final-answer persistence uses the completed action artifact as evidence.

### Scenario SCN-0007: Forbids final branch fields in action-only RLM payloads

Given an inference payload includes a top-level `final` field  
When strict schema validation runs  
Then the payload is rejected with typed output-validation behavior.

### Scenario SCN-0008: Rejects unknown fields under strict schema

Given inference payload contains unknown fields  
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

### Scenario SCN-0011: Exposes canonical and resolvable evidence references through context_ref and previous_action_feedback.action_ref

Given a step context with `context_ref` and a prior action with
`previous_action_feedback.action_ref`  
When evidence references are evaluated  
Then those refs are canonical and resolvable.

### Scenario SCN-0012: Validates marked final-answer action evidence before node completion

Given a completed action emits a marked final-answer block  
When node finalization runs  
Then the action artifact evidence ref is validated before completion.

### Scenario SCN-0013: Fails run with typed output-validation metadata when marked final-answer evidence cannot be resolved

Given marked final-answer evidence cannot be resolved  
When node finalization runs  
Then finalization fails with typed output-validation metadata.

### Scenario SCN-0014: Accepts final-answer evidence references for the canonical artifact scheme

Given marked final-answer evidence uses canonical `run-artifact://...` refs  
When evidence validation runs  
Then that scheme is accepted.

### Scenario SCN-0015: Persists enriched final-answer artifact with answer and action evidence

Given marked action finalization passes validation  
When final-answer persistence runs  
Then the persisted artifact contains answer and action evidence.

### Scenario SCN-0016: Requires byte-for-byte previous_action_feedback.action_ref reuse for action evidence recovery

Given REPL code recovers a prior action artifact  
When evidence refs are constructed or validated  
Then `previous_action_feedback.action_ref` is reused byte-for-byte instead of
synthesizing a new artifact ref.

### Scenario SCN-0017: Maintains inference schema_id sigil.rlm.response.v1 after action-only schema change

Given the structured response contract becomes action-only within this release  
When inference requests are built  
Then the request `schema_id` remains `sigil.rlm.response.v1`.
