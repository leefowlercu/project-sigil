# PRD-0015: Sigil Non-Timeout Stability and Observability Hardening Specification

## Status

Draft

## Context

`PRD-0007`, `PRD-0008`, `PRD-0009`, `PRD-0010`, `PRD-0012`, and `PRD-0013`
define event contracts, inference behavior, harness runtime, REPL execution,
bounded model-input rules, and subcall APIs. This PRD defines additive hardening
for non-timeout stability and failure visibility.

## Goals

- Define explicit failed-node terminal observability.
- Improve plain-subcall extraction robustness for `sigil.llm.answer.v1`.
- Persist structured compile diagnostics and feed them into next-step model
  input feedback.
- Preserve existing one-action-per-continue-step and subcall observability
  semantics.

## Non-Goals

- Changing CLI flags or run-config keys.
- Expanding `node.action.executed` payload schema with diagnostic detail.
- Introducing schema-agnostic raw-text fallback behavior.

## Node Failure Terminal Event Contract

- Runtime MUST define `node.failed` as canonical node terminal event for failed
  node executions.
- `node.failed` payload MUST include:
  - `status` literal `failed`
  - `duration_ms` integer `>= 0`
  - `error_code` non-empty string
  - `error_message` non-empty string
  - optional `failed_step_id` UUIDv7
- For each `node.started`, runtime MUST emit exactly one terminal node event:
  - `node.completed` OR
  - `node.failed`
- `node.completed` remains success-only and MUST NOT be used for failed nodes.

## Failure Ordering Contract

- Child failure ordering:
  - failing child node MUST emit `node.failed` before parent records failed
    `node.subcall.executed` item.
- Root failure ordering:
  - failing root node MUST emit `node.failed` before run emits `run.failed`.
- Child-node failure surfaced as `repl_child_failure` in parent continue action
  path remains non-fatal at run level unless escalation policy triggers fatal
  run failure.

## Plain-Subcall Extraction Fallback Contract

- Fallback behavior applies only when request schema is
  `sigil.llm.answer.v1`.
- If strict structured payload extraction fails and non-empty raw output text is
  available, adapter MUST normalize payload to:
  - `{"answer":"<trimmed raw text>"}`
- For all other schemas, extraction fallback MUST NOT run.
- When fallback path is used, normalized response `raw_metadata` MUST include
  extraction-mode metadata indicating fallback source and mode.

## Compile Diagnostics Artifact and Feedback Contract

- On `repl_execution_compile` failures, action artifact MAY include structured
  `error_detail`.
- `error_detail` shape:
  - `stage` literal `compile`
  - `message` required string
  - optional `line` integer
  - optional `column` integer
  - optional `symbol` string
  - optional `source_line` string
- Subsequent step `previous_action_feedback` MAY include matching
  `error_detail` when present in prior action artifact.
- Canonical `node.action.executed` payload contract remains unchanged and does
  not inline compile diagnostics.

## Failure Contract

- If node terminal failure event persistence fails, run MUST fail with typed
  infrastructure metadata.
- If compile diagnostics parsing fails, existing `error_code`/`error_message`
  behavior remains authoritative.

## Deferred Contracts

- Structured diagnostics for runtime/panic failures.
- Cross-schema raw-text normalization policies.
- Extended node terminal analytics payloads beyond v1 fields.

## Acceptance Scenarios

### Scenario SCN-0000: Defines node.failed as canonical node terminal event for failed node executions

Given canonical v1 runtime event contracts  
When a node execution terminates with failure  
Then node terminal event is emitted as `node.failed`.

### Scenario SCN-0001: Emits exactly one terminal node event node.completed or node.failed for every node.started

Given one or more `node.started` events in a run  
When node executions reach terminal states  
Then each started node has exactly one terminal node event of `node.completed`
or `node.failed`.

### Scenario SCN-0002: Emits node.failed for recursive child execution failures before parent node.subcall.executed failed record

Given recursive child execution fails  
When failure events are persisted  
Then child `node.failed` is emitted before parent failed
`node.subcall.executed` record.

### Scenario SCN-0003: Emits node.failed for root execution failures before run.failed

Given root node execution fails  
When terminal failure events are persisted  
Then `node.failed` is emitted before `run.failed`.

### Scenario SCN-0004: Preserves run continuation when child node.failed is surfaced as repl_child_failure in parent continue action path

Given child node failure is surfaced as `repl_child_failure` in parent continue
path  
When parent node handles non-fatal action failure  
Then parent node can continue to next decision step.

### Scenario SCN-0005: Applies schema-specific raw-text fallback for sigil.llm.answer.v1 when strict structured payload extraction fails

Given inference request schema is `sigil.llm.answer.v1` with non-empty raw text
output and failed strict extraction  
When adapter normalizes structured payload  
Then payload is normalized to strict answer-object shape using trimmed raw text.

### Scenario SCN-0006: Rejects raw-text fallback for non sigil.llm.answer.v1 schemas

Given inference request schema is not `sigil.llm.answer.v1` and strict
structured extraction fails  
When adapter handles extraction failure  
Then raw-text fallback is not applied and extraction fails.

### Scenario SCN-0007: Emits extraction-mode metadata in normalized inference raw_metadata for plain-subcall responses

Given plain-subcall extraction fallback path is used  
When normalized inference result is emitted  
Then `raw_metadata` includes extraction-mode metadata indicating fallback mode
and source.

### Scenario SCN-0008: Persists structured compile diagnostics in failed action artifacts for repl_execution_compile errors

Given continue action fails with `repl_execution_compile`  
When action artifact is persisted  
Then artifact includes structured compile `error_detail` when parseable.

### Scenario SCN-0009: Includes compile diagnostics in previous_action_feedback for subsequent model steps

Given prior continue action artifact contains compile `error_detail`  
When next-step model-input feedback is built  
Then `previous_action_feedback.error_detail` is included.

### Scenario SCN-0010: Preserves node.action.executed payload contract while exposing diagnostics through artifact and feedback only

Given compile diagnostics hardening is active  
When `node.action.executed` event is emitted  
Then payload contract remains unchanged and diagnostics are exposed only through
artifact and feedback schemas.

### Scenario SCN-0011: Preserves one-action-per-continue-step and subcall observability contracts under hardening changes

Given failure hardening features are active  
When continue steps execute with or without subcalls  
Then one-action-per-continue-step and subcall observability contracts are
unchanged.
