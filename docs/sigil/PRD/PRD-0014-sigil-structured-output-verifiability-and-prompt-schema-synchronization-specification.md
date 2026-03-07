# PRD-0014: Sigil Structured Output Verifiability and Prompt-Schema Synchronization Specification

## Status

Draft

## Context

`PRD-0008`, `PRD-0009`, `PRD-0011`, and `PRD-0012` define core inference,
harness execution, run-start orchestration, and bounded model-input behavior.
This PRD defines Priority #3 hard-cutover contracts for enriched structured
outputs, strict evidence resolution, and runtime prompt-schema synchronization.

## Goals

- Strengthen `sigil.rlm.response.v1` for verifiable execution and final answers.
- Require explicit continuation intent metadata.
- Require final evidence references and optional confidence signaling.
- Ensure first-step final responses can cite resolvable context evidence.
- Eliminate prompt/schema drift by deriving prompt schema text from registry.

## Non-Goals

- Backward compatibility shims for legacy v1 payload shape.
- User-configurable evidence validation modes.
- Cross-run evidence resolution.

## Structured Output Contract

`sigil.rlm.response.v1` MUST enforce:

- `decision=continue`:
  - `continuation.repl_code` (required, non-empty)
  - `continuation.intent` (required, non-empty)
  - `continuation.expected_observation` (required, non-empty)
  - strict unknown-field rejection
- `decision=final`:
  - `final.answer` (required, non-empty)
  - `final.evidence` (required array, minItems=1)
  - `final.confidence` (optional enum: `low|medium|high`)
  - strict unknown-field rejection

Evidence item shape:

- `ref` (required string, non-empty)
- `chunk_id` (optional string)
- `span_start` (optional integer >= 0)
- `span_end` (optional integer >= 0; when `span_start` exists, `span_end` MUST
  be `>= span_start`)
- unknown fields are rejected

## Context Reference Contract

- Harness MUST persist one node context artifact at:
  - `run-output://node/<node_id>/context.json`
- `context_metadata` MUST include `context_ref`.
- `context_ref` MUST be non-empty and resolvable in run-local storage.

## Evidence Resolution Contract

- `final.evidence[].ref` MUST resolve before node completion.
- Accepted canonical schemes in v1:
  - `run-output://...`
  - `run-artifact://...`
- When final evidence cites a prior action artifact surfaced via
  `previous_action_feedback.output_ref`, the cited ref MUST match the provided
  `output_ref` byte-for-byte.
- If exact reuse of `previous_action_feedback.output_ref` is not possible,
  final evidence MUST cite `context_ref` instead of synthesizing a new
  `run-artifact://...` ref.
- Unresolvable refs MUST fail finalization as typed output-validation failure.

## Final Output Persistence Contract

- Final-answer artifact MUST persist:
  - `answer`
  - `evidence[]`
  - optional `confidence`
- `run.completed.final_answer_ref` continues to point at persisted final-answer
  artifact.

## Prompt-Schema Synchronization Contract

- Provider base prompts MUST live in
  `sigil/internal/harness/system_prompts.go`.
- System prompt schema block MUST be rendered at runtime from
  `schema.SigilRLMResponseV1SchemaID` definition.
- Effective prompt resolution (`provider` mapping + `system_prompt_append`)
  occurs after runtime schema rendering.

## Compatibility Contract

- Continue-step one-action-per-step behavior is unchanged.
- Recursive/non-recursive execution profiles are unchanged.
- Inference request `schema_id` remains `sigil.rlm.response.v1`.

## Acceptance Scenarios

### Scenario SCN-0000: Requires continuation intent expected_observation and repl_code in continue branch

Given an inference payload with `decision=continue`  
When strict schema validation runs  
Then `continuation.repl_code`, `continuation.intent`, and
`continuation.expected_observation` are required and non-empty.

### Scenario SCN-0001: Requires final answer evidence and optional confidence enum in final branch

Given an inference payload with `decision=final`  
When strict schema validation runs  
Then `final.answer` and `final.evidence` are required and
`final.confidence` is optional with enum `low|medium|high`.

### Scenario SCN-0002: Rejects unknown fields and malformed evidence entries under strict schema

Given inference payload containing unknown fields or invalid evidence span data  
When strict schema validation runs  
Then payload is rejected with typed output-validation behavior.

### Scenario SCN-0003: Persists node context artifact and includes context_ref in step input context_metadata

Given an executing node with resolved context  
When step input metadata is built  
Then node context artifact is persisted and `context_metadata.context_ref` is
present.

### Scenario SCN-0004: Exposes canonical and resolvable evidence references through context_ref and previous_action_feedback.output_ref

Given a model step envelope and subsequent feedback envelope  
When evidence-capable references are inspected  
Then `context_ref` and `previous_action_feedback.output_ref` are canonical and
resolvable.

### Scenario SCN-0005: Validates final evidence references against run-local persisted artifacts before node completion

Given a final decision payload with evidence references  
When harness finalizes node completion  
Then every evidence ref is validated against run-local persisted artifacts.

### Scenario SCN-0006: Fails run with typed output-validation metadata when any final evidence reference cannot be resolved

Given a final decision payload containing at least one unresolved evidence ref  
When node finalization runs  
Then run fails with typed output-validation metadata.

### Scenario SCN-0007: Accepts final evidence references for canonical run-output and run-artifact schemes

Given a final decision payload with evidence refs in canonical schemes  
When evidence resolution runs  
Then `run-output://...` and `run-artifact://...` refs are accepted when
resolvable.

### Scenario SCN-0008: Persists enriched final-answer artifact with answer evidence and optional confidence

Given a resolved final decision payload  
When final-answer persistence runs  
Then persisted final artifact includes `answer`, `evidence[]`, and optional
`confidence`.

### Scenario SCN-0009: Generates system prompt schema block from central registry definition sigil.rlm.response.v1 at runtime

Given system prompt construction for an inference step  
When schema block rendering executes  
Then schema block is generated from registry definition
`sigil.rlm.response.v1` at runtime.

### Scenario SCN-0010: Applies provider prompt resolution and system_prompt_append composition after runtime schema rendering

Given configured provider and optional `system_prompt_append`  
When effective system prompt is resolved  
Then provider mapping and append composition are applied after runtime schema
rendering.

### Scenario SCN-0011: Preserves one-action-per-continue-step and recursion profile semantics under enriched schema

Given enriched structured response payloads are active  
When continue and recursive/non-recursive execution run  
Then one-action-per-continue-step and recursion profile semantics remain
unchanged.

### Scenario SCN-0012: Maintains inference schema_id sigil.rlm.response.v1 after schema extension

Given enriched schema contract is active  
When inference request is constructed  
Then request schema_id remains `sigil.rlm.response.v1`.

### Scenario SCN-0013: Maintains prompt-schema parity by deriving prompt schema text from the same registry definition used for strict inference validation

Given strict inference validation and prompt generation are active  
When schema sources are compared  
Then prompt schema text is derived from the same registry definition used for
strict validation.

### Scenario SCN-0014: Requires byte-for-byte previous_action_feedback.output_ref reuse with context_ref fallback for final evidence citations

Given final evidence cites a prior continue-step action artifact  
When effective system prompt instructions are inspected  
Then prompt instructions require exact `previous_action_feedback.output_ref`
reuse and `context_ref` fallback instead of synthesized `run-artifact` refs.
