# PRD-0010: Sigil Go REPL Runtime Specification

## Status

Draft

## Context

`PRD-0009` defines harness-level recursive control behavior for `sigil`, but the
runtime contract for executing `continuation.repl_code` inside a node-local Go
REPL environment requires a dedicated behavior specification.

This PRD defines v1 REPL runtime behavior including session lifecycle,
recursive REPL bindings, fixed guardrails, artifact persistence, and error
policy.

## Goals

- Define v1 Go REPL runtime behavior for `continuation.repl_code` actions.
- Define node-local REPL session lifecycle and persistence rules.
- Define REPL API surface including `context` and subcall query functions.
- Define fixed v1 execution guardrails and import policy.
- Define artifact persistence and `output_ref` reference behavior.
- Define typed REPL runtime error taxonomy and failure policy.

## Non-Goals

- Full sandbox hardening (for example seccomp/cgroups/syscall-level controls).
- Multi-language REPL support.
- User-configurable REPL limits in v1.

## Session Lifecycle Contract

- REPL runtime MUST maintain exactly one session per node ID.
- Node REPL session MUST be created lazily on first continue action for that
  node.
- Node REPL session state MUST persist across all continue steps of that node.
- Node REPL session MUST close when node completes.
- Remaining REPL sessions MUST close when run reaches any terminal state.

## REPL API Contract

- REPL session MUST expose `context` as a string value.
- REPL session MUST expose:
  - `llm_query(prompt string, context string) (string, error)`
  - `rlm_query(prompt string, context string) (string, error)`
- `llm_query` MUST execute one plain subcall inference and MUST NOT create a
  child node.
- REPL session MUST expose:
  - `llm_query_batched(calls []map[string]string) ([]map[string]string, error)`
  - `rlm_query_batched(calls []map[string]string) ([]map[string]string, error)`
- Batched call items MUST use keys:
  - `prompt`
  - `context`
- Batched result items MUST include keys:
  - `answer`
  - `error_code`
  - `error_message`
- `rlm_query` MUST:
  - create a child node when `parent_depth + 1 <= rlm.max_depth`
  - return child `final.answer` on successful child completion
  - execute recursive subcalls with run-scoped context and independent
    recursive timeout budget (not parent action elapsed deadline and not
    ancestor recursive subcall deadline depletion)
  - fallback to plain `llm_query` behavior when depth limit is reached in
    recursive profile
  - return typed runtime error on child-node failure propagation
- `rlm_query_batched` MUST:
  - execute sequentially in input order
  - execute recursive subcalls with run-scoped context and independent
    recursive timeout budget (not parent action elapsed deadline and not
    ancestor recursive subcall deadline depletion)
  - fallback per item to plain subcall behavior when depth limit is reached in
    recursive profile
  - return typed depth-limit behavior for all items when non-recursive profile
    is active.

## Execution Model Contract

- `decision=continue` MUST execute exactly one `continuation.repl_code` action
  in the current node session.
- Every continue action execution MUST emit `node.action.executed`.
- Node step event ordering for continue steps MUST be:
  1. `node.step.started`
  2. `node.turn.user`
  3. `node.turn.model`
  4. zero or more `node.subcall.executed`
  5. `node.action.executed`
  6. `node.step.completed`

## Fixed v1 Guardrails

- Action timeout MUST be `180s`.
- Recursive subcall timeout MUST be `300s`.
- Maximum `repl_code` payload size MUST be `65536` bytes.
- Captured stdout cap MUST be `1048576` bytes.
- Captured stderr cap MUST be `1048576` bytes.
- Output exceeding caps MUST be truncated with deterministic truncation marker.

## Import Policy Contract

- V1 MUST enforce allowlist-only import policy.
- Allowed stdlib packages in v1 are:
  - `fmt`
  - `strings`
  - `strconv`
  - `sort`
  - `regexp`
  - `encoding/json`
  - `bytes`
  - `math`
  - `time`
  - `slices`
- OS/network/process-sensitive packages (for example `os`, `os/exec`, `net`,
  `syscall`) MUST be blocked.

## Artifact and Reference Contract

- Every action execution MUST persist one artifact JSON document under run-local
  storage.
- `node.action.executed.output_ref` MUST be present for both successful and
  failed action executions.
- `output_ref` MUST resolve to the persisted action artifact for the same
  `run_id`, `node_id`, `step_id`, and `action_index`.
- V1 canonical reference format MUST be:
  - `run-artifact://node/<node_id>/step/<step_id>/action-<action_index>.json`
- V1 canonical storage path mapped from this reference MUST be:
  - `./.sigil/runs/<run_id>/artifacts/node/<node_id>/step/<step_id>/action-<action_index>.json`
- Artifact payload MUST include:
  - run/node/step/action identity fields
  - `action_type`, `language`, and execution `status`
  - raw `repl_code`
  - captured stdout/stderr (possibly truncated)
  - execution duration
  - typed error metadata when failed
- On compile-stage failures, artifacts MAY include structured `error_detail`
  with:
  - `stage` literal `compile`
  - required `message`
  - optional `line`, `column`, `symbol`, `source_line`

## Error Policy Contract

- Non-fatal REPL execution failures (`compile`, `runtime`, `timeout`,
  `import_policy`, `code_size`) MUST:
  - set `node.action.executed.status=failed`
  - carry deterministic typed error metadata in event and artifact
  - feed deterministic error feedback into subsequent model context
  - include compile-stage structured diagnostics in subsequent-step feedback
    when parseable
  - continue node execution to the next decision step
- Fatal infrastructure failures (`session initialization`, `artifact
  persistence`, internal runtime corruption/panic) MUST fail the run with typed
  error metadata.

## Typed Error Taxonomy

REPL runtime MUST use machine-readable error codes including:

- `repl_session_init`
- `repl_code_size_exceeded`
- `repl_import_blocked`
- `repl_execution_timeout`
- `repl_execution_compile`
- `repl_execution_runtime`
- `repl_artifact_persist`
- `repl_child_depth_limit`
- `repl_child_failure`
- `repl_subcall_invalid_input`
- `repl_subcall_inference`
- `repl_subcall_event_persist`

## Internal Interface Contract

V1 REPL runtime boundary MUST document and preserve these internal contracts:

- `Session` with:
  - `Exec(ctx context.Context, code string) (ExecResult, error)`
  - `Close() error`
- `SessionFactory` with:
  - node-scoped session creation for a given node identity and bindings
- `llm_query(prompt, context) (string, error)` REPL binding as specified above.
- `rlm_query(prompt, context) (string, error)` REPL binding as specified above.
- batched REPL bindings:
  - `llm_query_batched(calls []map[string]string) ([]map[string]string, error)`
  - `rlm_query_batched(calls []map[string]string) ([]map[string]string, error)`

## Deferred Contracts

The following are deferred to future PRDs:

- Out-of-process REPL execution backends.
- Per-run configurable REPL limits and policies.
- Fine-grained per-package capability profiles.
- REPL telemetry streaming beyond persisted artifacts.

## Acceptance Scenarios

### Scenario SCN-0000: Selects embedded Go REPL engine architecture for v1

Given v1 REPL runtime architecture rules  
When REPL engine configuration is resolved  
Then embedded in-process Go interpretation is selected.

### Scenario SCN-0001: Creates exactly one persistent REPL session per node

Given an active node with no existing REPL session  
When first continue action executes  
Then one REPL session is created and associated to that node.

### Scenario SCN-0002: Reuses node REPL state across multiple continue steps

Given an active node with existing REPL session state  
When additional continue actions execute for that node  
Then subsequent actions run in the same node REPL session state.

### Scenario SCN-0003: Closes node REPL session on node completion and run termination

Given active node REPL sessions exist  
When node completes or run enters terminal state  
Then corresponding REPL sessions are closed.

### Scenario SCN-0004: Executes continuation.repl_code in node-local REPL session

Given a continue step with non-empty continuation.repl_code  
When harness executes action handling  
Then continuation.repl_code executes in the current node-local REPL session.

### Scenario SCN-0005: Exposes llm_query rlm_query llm_query_batched and rlm_query_batched in node-local REPL session

Given a node-local REPL session is initialized  
When REPL bindings are inspected  
Then llm_query rlm_query llm_query_batched and rlm_query_batched are available.

### Scenario SCN-0006: Creates child node for rlm_query when depth is within limit

Given parent node depth and rlm.max_depth permit recursion  
When rlm_query is invoked  
Then child node is created and executed.

### Scenario SCN-0007: Falls back to llm_query when rlm_query reaches max depth in recursive mode

Given parent node depth equals rlm.max_depth  
When rlm_query is invoked  
Then plain llm_query fallback behavior is used and child node is not created.

### Scenario SCN-0008: Returns child final answer to caller REPL context on successful subcall

Given child node reaches final decision with answer  
When rlm_query completes  
Then child final answer is returned to caller REPL context.

### Scenario SCN-0009: Records failed action and continues next step on non-fatal REPL execution error

Given a continue action fails with non-fatal REPL execution error  
When action failure is handled  
Then action failure is recorded and node execution continues to next step.

### Scenario SCN-0010: Enforces 180-second execution timeout per action

Given a continue action exceeding 180 seconds execution time  
When REPL runtime enforces guardrails  
Then action times out with typed timeout error.

### Scenario SCN-0011: Rejects repl_code payloads larger than 65536 bytes

Given a continue action with repl_code payload larger than 65536 bytes  
When payload guardrails are validated  
Then action is rejected with typed code-size error.

### Scenario SCN-0012: Truncates stdout/stderr deterministically at 1048576-byte caps

Given an action execution producing stdout or stderr over 1048576 bytes  
When output capture guardrails are enforced  
Then outputs are truncated with deterministic truncation marker.

### Scenario SCN-0013: Enforces allowlist-only import policy and rejects blocked imports

Given continue action code imports blocked packages  
When REPL import policy validation executes  
Then action fails with typed import-blocked error.

### Scenario SCN-0014: Persists per-action artifact and sets node.action.executed.output_ref

Given an action execution completes or fails  
When action artifact persistence executes  
Then artifact is persisted and node.action.executed.output_ref is set to canonical artifact reference.

### Scenario SCN-0015: Fails run on fatal REPL infrastructure errors with typed error metadata

Given fatal REPL infrastructure failure occurs  
When harness handles failure propagation  
Then run transitions to failed with typed error metadata.

### Scenario SCN-0016: Returns structured per-item batched results for llm_query_batched and rlm_query_batched

Given batched subcall execution includes mixed item outcomes  
When llm_query_batched or rlm_query_batched returns  
Then each result item includes answer error_code and error_message fields.

### Scenario SCN-0017: Emits node.subcall.executed for each subcall item executed inside continue action

Given a continue action executes one or more subcall items  
When runtime events are persisted  
Then one node.subcall.executed event exists per subcall item.

### Scenario SCN-0018: Executes recursive subcalls with independent 300-second timeout budget decoupled from parent and recursive-level elapsed deadlines

Given recursive subcalls execute from an active continue action  
When recursive subcall contexts are constructed  
Then each recursive subcall gets an independent 300-second timeout budget from run-scoped context without inheriting ancestor recursive subcall deadline depletion.

### Scenario SCN-0019: Cancels recursive subcalls on run-context cancellation despite timeout decoupling from parent action context

Given recursive subcalls execute with timeout decoupled from parent action elapsed time  
When run context is canceled  
Then in-flight recursive subcalls are canceled deterministically.

### Scenario SCN-0020: Persists structured compile diagnostics in failed action artifacts for repl_execution_compile errors

Given a continue action fails with `repl_execution_compile`  
When action artifact persistence executes  
Then artifact includes structured compile diagnostics when parseable.

### Scenario SCN-0021: Propagates compile diagnostics into previous-action feedback for subsequent model steps

Given prior continue action artifact contains structured compile diagnostics  
When next-step feedback is built  
Then previous-action feedback includes compile diagnostic detail.
