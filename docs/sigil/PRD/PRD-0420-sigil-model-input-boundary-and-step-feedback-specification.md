# PRD-0420: Sigil Model Input Boundary and Step Feedback Specification

## Status

Draft

## Context

`sigil` keeps full execution context in REPL scope while sending bounded,
deterministic step input to the model. That boundary needs one normative owner
so prompt and feedback growth does not collapse into replaying raw runtime
state.

This PRD owns:

- bounded model-input envelope construction
- step-level `user` message shape
- context metadata and `context_ref`
- `execution_state` progress metadata
- `previous_action_feedback` shape
- compact user-turn artifact behavior

## Goals

- Exclude full raw context from model-step inference requests.
- Define deterministic step-envelope content and bounded feedback behavior.
- Expose deterministic execution-state progress metadata to model-step inference.
- Define compact `node.turn.user` artifact behavior.
- Preserve recursive and non-recursive profile behavior under the bounded-input contract.

## Non-Goals

- Changing structured response schemas owned by `PRD-0310`.
- Changing action artifact persistence semantics owned by `PRD-0430`.
- Defining inference retry or transport behavior owned by `PRD-0300`.
- Defining user-configurable preview caps in this release.

## Model-Input Boundary Contract

- Full raw run `context` MUST remain available to REPL execution.
- Harness MUST NOT replay full raw context into model-step inference messages.
- This boundary applies equally to root and recursive child nodes.

## Message Contract

- Harness inference requests MUST use ordered role-based `messages`.
- For each step, message order MUST be:
  1. `system`
  2. `user`
- The `user` message MUST be deterministic JSON containing:
  - `query`
  - `step_index`
  - `context_metadata`
  - `execution_state`
  - optional `previous_action_feedback`

## Context Metadata Contract

`context_metadata` MUST include:

- `context_type` with string literal `string`
- `context_bytes`
- `context_line_count`
- `context_sha256`
- `context_ref`

`context_ref` rules:

- `context_ref` MUST be a canonical run-output reference.
- `context_ref` MUST be non-empty and resolvable in run-local storage.
- The referenced artifact is the canonical persisted form of node-local context for evidence and prompt-boundary workflows.

## Execution State Contract

`execution_state` MUST include:

- `node_depth`
- `max_depth`
- `remaining_depth`
- `node_steps_used`
- `node_steps_remaining`
- `run_steps_used`
- `run_steps_remaining`
- `same_context_as_previous_step`
- `small_context`
- `recursive_subcalls_allowed`
- optional `recursive_subcalls_reason`

`execution_state` rules:

- values MUST be deterministic for equivalent node state and guardrail budgets
- `same_context_as_previous_step` MUST be `false` on the first step and `true` on later steps of the same node
- `small_context` MUST be derived from deterministic context-size heuristics rather than model output
- `recursive_subcalls_reason`, when present, MUST explain why recursive subcalls are not allowed for the current step

## Previous Action Feedback Contract

- First step MUST omit `previous_action_feedback`.
- For subsequent steps after continue actions, feedback MUST include:
  - `output_ref`
  - `status`
  - optional `error_code`
  - optional `error_message`
  - optional `error_detail`
  - optional `subcall_summary`
  - `stdout_preview`
  - `stdout_bytes`
  - `stdout_truncated`
  - `stderr_preview`
  - `stderr_bytes`
  - `stderr_truncated`
- Preview caps are fixed in this release:
  - `stdout_preview <= 2048 bytes`
  - `stderr_preview <= 2048 bytes`
- `error_detail`, when present, MUST describe compile-stage diagnostics using:
  - `stage` literal `compile`
  - required `message`
  - optional `line`
  - optional `column`
  - optional `symbol`
  - optional `source_line`
- `subcall_summary`, when present, MUST include deterministic counts for:
  - total subcalls
  - plain-mode subcalls
  - recursive-mode subcalls
  - fallback-mode subcalls
  - completed subcalls
  - failed subcalls
- Action artifacts remain the source of truth for full stdout and stderr.
- Exact action output recovery, when needed, MUST occur through REPL-side `read_action_output(output_ref)` rather than widening `previous_action_feedback`.

## User-Turn Artifact Contract

- `node.turn.user.content_ref` MUST resolve to a compact model-input artifact.
- Compact user-turn artifacts MUST include:
  - run, node, and step identity
  - deterministic step-envelope fields
  - model-input message role metadata
- Compact user-turn artifacts MUST NOT embed the full raw context body.

## Failure Contract

- If step-envelope encoding or persistence fails, the run MUST fail with typed infrastructure metadata.
- Canonical event ordering from `PRD-0210` MUST remain unchanged.

## Deferred Contracts

- Dynamic feedback cap tuning by run config
- additional context metadata shapes for non-string context types
- richer feedback channels beyond bounded previews and compile diagnostics
- exposing full action artifact payloads directly in `previous_action_feedback`

## Acceptance Scenarios

### Scenario SCN-0000: Keeps full raw context in REPL scope and excludes it from model-step inference messages

Given an active node with raw context initialized in REPL scope  
When model-step inference messages are constructed  
Then full raw context is excluded from outbound model-step inference messages.

### Scenario SCN-0001: Builds model-step inference input as ordered role-based messages system then user

Given an active node step  
When harness constructs inference input  
Then message order is `system` then `user`.

### Scenario SCN-0002: Constructs deterministic user step envelope with query step index and context metadata

Given an active node step with resolved query and context  
When the user step envelope is encoded  
Then the envelope contains deterministic `query`, `step_index`, and `context_metadata` fields.

### Scenario SCN-0003: Includes bounded previous-action feedback summary with output_ref and preview truncation metadata

Given a node step after a continue action has executed  
When the user step envelope is encoded  
Then `previous_action_feedback` includes `output_ref` and bounded preview truncation metadata.

### Scenario SCN-0004: Omits previous-action feedback block on first step before any continue action executes

Given the first node step before any continue action  
When the user step envelope is encoded  
Then `previous_action_feedback` is omitted.

### Scenario SCN-0005: Persists compact node.turn.user artifact without embedding full raw context

Given node turn user output persistence executes  
When the compact artifact is written  
Then the artifact excludes full raw context body and preserves compact metadata plus refs.

### Scenario SCN-0006: Preserves action artifact as source of truth for full stdout and stderr while model receives bounded previews

Given a continue action captures stdout and stderr  
When subsequent step input is built  
Then the model receives bounded previews and the action artifact remains the full source of truth.

### Scenario SCN-0007: Constructs OpenRouter Responses API requests with message-array input preserving role order

Given bounded step-input construction completes  
When inference request input is passed to the gateway layer  
Then the request preserves ordered role-based message-array input.

### Scenario SCN-0008: Applies bounded model-input contract consistently to recursive child nodes

Given recursive child-node execution is active  
When model-step input is constructed for the child node  
Then the same bounded model-input contract applies.

### Scenario SCN-0009: Preserves non-recursive profile behavior under bounded model-input contract

Given `rlm.enabled=false` and active multi-step execution  
When model-step input is constructed across steps  
Then bounded model-input behavior remains unchanged.

### Scenario SCN-0010: Fails run with typed infrastructure metadata when step-envelope serialization or persistence fails

Given step-envelope serialization or persistence fails  
When the boundary layer handles the failure  
Then the run fails with typed infrastructure metadata.

### Scenario SCN-0011: Maintains canonical run event ordering and references under bounded model-input execution

Given bounded model-input execution is active  
When step and turn events are persisted  
Then canonical event ordering and reference behavior remain unchanged.

### Scenario SCN-0012: Persists context artifact reference in context_metadata for model-step envelopes

Given node context artifact persistence executes  
When step input is constructed  
Then `context_metadata.context_ref` contains the persisted context artifact reference.

### Scenario SCN-0013: Includes optional previous_action_feedback.error_detail for compile-stage failures

Given the prior continue action failed at compile stage  
And structured compile diagnostics are available in the action artifact  
When subsequent step input is constructed  
Then `previous_action_feedback.error_detail` is included.

### Scenario SCN-0014: Includes execution_state with depth step budgets and recursion-permission metadata in user step envelope

Given an active node step with resolved query context and guardrail budgets  
When the user step envelope is encoded  
Then `execution_state` includes depth step-budget and recursion-permission metadata.

### Scenario SCN-0015: Includes previous_action_feedback.subcall_summary with deterministic counts by execution mode and status

Given a prior continue action executed one or more subcalls  
When subsequent step input is constructed  
Then `previous_action_feedback.subcall_summary` includes deterministic counts by execution mode and status.

### Scenario SCN-0016: Preserves bounded previous_action_feedback previews while exact action output remains recoverable through read_action_output

Given a node step after a continue action has executed  
When bounded feedback is exposed to the model and REPL  
Then previews remain bounded and exact action output remains recoverable through `read_action_output(output_ref)`.
