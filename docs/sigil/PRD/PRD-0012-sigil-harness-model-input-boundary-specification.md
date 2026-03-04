# PRD-0012: Sigil Harness Model-Input Boundary Specification

## Status

Draft

## Context

`PRD-0009`, `PRD-0010`, and `PRD-0011` define harness control flow, Go REPL
runtime behavior, and blocking `run start` execution. This PRD specifies the
Priority 1 model-input boundary that keeps full context in REPL scope and sends
bounded per-step model input.

## Goals

- Exclude full raw context from model-step inference requests.
- Move inference request construction to ordered role-based messages.
- Define deterministic step envelope content and bounded feedback behavior.
- Define compact `node.turn.user` artifact behavior.
- Preserve recursive and non-recursive profile behavior under bounded input.

## Non-Goals

- Changing `sigil.rlm.response.v1`.
- Adding user-configurable feedback preview caps in v1.
- Changing action artifact persistence semantics in `PRD-0010`.

## Model-Input Boundary Contract

- Full raw run `context` MUST remain available to REPL execution.
- Harness MUST NOT replay full raw context into model-step inference messages.
- This boundary applies equally to root and recursive child nodes.

## Message Contract

- Harness inference requests MUST use ordered role-based `messages`.
- For each step, message order MUST be:
  1. `system`
  2. `user`
- `user` message MUST be deterministic JSON containing:
  - `query`
  - `step_index`
  - `context_metadata`
  - optional `previous_action_feedback`

## Context Metadata Contract

`context_metadata` in v1 MUST include:

- `context_type` (string literal `string`)
- `context_bytes` (integer bytes of raw context)
- `context_line_count` (integer line count)
- `context_sha256` (hex SHA-256 of raw context)
- `context_ref` (canonical run-output reference to persisted node context
  artifact)

## Previous Action Feedback Contract

- First step MUST omit `previous_action_feedback`.
- For subsequent steps after continue actions, feedback block MUST include:
  - `output_ref`
  - `status`
  - optional `error_code`
  - optional `error_message`
  - `stdout_preview`
  - `stdout_bytes`
  - `stdout_truncated`
  - `stderr_preview`
  - `stderr_bytes`
  - `stderr_truncated`
- Preview caps in v1 are fixed:
  - `stdout_preview` <= 2048 bytes
  - `stderr_preview` <= 2048 bytes
- Action artifact remains source of truth for full stdout/stderr.

## User-Turn Artifact Contract

- `node.turn.user` `content_ref` MUST resolve to compact model-input artifact.
- Compact user-turn artifact MUST include:
  - run/node/step identity
  - deterministic step envelope fields
  - model-input message role metadata
- Compact user-turn artifact MUST NOT embed full raw context body.

## Failure Contract

- If step-envelope encoding/persistence fails, run MUST fail with typed
  infrastructure metadata.
- Canonical event ordering from `PRD-0007` MUST remain unchanged.

## Deferred Contracts

- Dynamic feedback cap tuning by run config.
- Additional context metadata shapes for non-string context types.

## Acceptance Scenarios

### Scenario SCN-0000: Keeps full raw context in REPL scope and excludes it from model-step inference messages

Given an active node with raw context initialized in REPL scope  
When model-step inference messages are constructed  
Then full raw context is excluded from outbound model-step inference messages.

### Scenario SCN-0001: Builds model-step inference input as ordered role-based messages system then user

Given an active node step  
When harness constructs inference input  
Then message list is ordered as system then user.

### Scenario SCN-0002: Constructs deterministic user step envelope with query step index and context metadata

Given an active node step with resolved query and context  
When user step envelope is encoded  
Then envelope contains deterministic query step_index and context_metadata fields.

### Scenario SCN-0003: Includes bounded previous-action feedback summary with output_ref and preview truncation metadata

Given a node step after a continue action has executed  
When user step envelope is encoded  
Then previous_action_feedback includes output_ref and bounded preview truncation metadata.

### Scenario SCN-0004: Omits previous-action feedback block on first step before any continue action executes

Given first node step before any continue action  
When user step envelope is encoded  
Then previous_action_feedback is omitted.

### Scenario SCN-0005: Persists compact node.turn.user artifact without embedding full raw context

Given node turn user output persistence executes  
When compact artifact is written  
Then artifact excludes full raw context body and preserves compact metadata plus refs.

### Scenario SCN-0006: Preserves action artifact as source of truth for full stdout and stderr while model receives bounded previews

Given a continue action with captured stdout and stderr  
When next-step feedback is prepared  
Then model input receives bounded previews and action artifact remains full-output source of truth.

### Scenario SCN-0007: Constructs OpenRouter Responses API requests with message-array input preserving role order

Given a valid inference request  
When OpenRouter request payload is constructed  
Then request uses message-array input preserving role order.

### Scenario SCN-0008: Applies bounded model-input contract consistently to recursive child nodes

Given recursive child node execution  
When child step inference input is constructed  
Then bounded model-input contract is applied identically to child node steps.

### Scenario SCN-0009: Preserves non-recursive profile behavior under bounded model-input contract

Given non-recursive harness mode is active  
When model-step inference input is constructed across multiple steps  
Then bounded model-input behavior is preserved and non-recursive execution semantics remain unchanged.

### Scenario SCN-0010: Fails run with typed infrastructure metadata when step-envelope serialization or persistence fails

Given step-envelope serialization or persistence failure  
When harness handles failure propagation  
Then run fails with typed infrastructure metadata.

### Scenario SCN-0011: Maintains canonical run event ordering and references under bounded model-input execution

Given bounded model-input execution is active  
When step and turn events are persisted  
Then canonical event ordering and reference invariants remain valid.

### Scenario SCN-0012: Persists context artifact reference in context_metadata for model-step envelopes

Given an active node with effective context  
When deterministic step input envelope is built  
Then `context_metadata.context_ref` is present and resolves to persisted
run-output context artifact.
