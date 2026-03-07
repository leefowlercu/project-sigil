# PRD-0430: Sigil Go REPL Runtime Specification

## Status

Draft

## Context

`sigil` executes continuation actions inside a node-local Go REPL environment.
That runtime needs a dedicated behavior specification separate from the harness
control loop and separate from the subcall API surface.

This PRD owns:

- REPL session lifecycle
- continuation execution behavior
- fixed execution limits
- import policy
- action artifact persistence
- compile-diagnostics artifact behavior
- typed REPL runtime failures

Subcall API behavior is defined in `PRD-0440`. Step-input feedback behavior is
defined in `PRD-0420`.

## Goals

- Define node-local Go REPL runtime behavior for `continuation.repl_code`.
- Define REPL session lifecycle and persistence rules.
- Define fixed execution limits and import policy.
- Define artifact persistence and `output_ref` behavior.
- Define compile-diagnostics persistence and typed runtime failures.

## Non-Goals

- Full sandbox hardening beyond the fixed import policy and limits
- multi-language REPL support
- user-configurable REPL limits in this release
- defining recursive subcall API semantics

## Session Lifecycle Contract

- REPL runtime MUST maintain exactly one session per node ID.
- A node REPL session MUST be created lazily on first continue action for that node.
- Node REPL session state MUST persist across all continue steps of that node.
- Node REPL session MUST close when the node completes.
- Remaining REPL sessions MUST close when the run reaches any terminal state.

## Continuation Execution Contract

- `decision=continue` MUST execute exactly one `continuation.repl_code` action in the current node session.
- Every continue action execution MUST emit `node.action.executed`.
- Non-fatal REPL execution failures MUST record a failed action and allow the node to continue to the next decision step.

## Fixed Limits Contract

- Action timeout MUST be `180s`.
- Maximum `repl_code` payload size MUST be `65536` bytes.
- Captured stdout cap MUST be `1048576` bytes.
- Captured stderr cap MUST be `1048576` bytes.
- Output exceeding caps MUST be truncated with a deterministic truncation marker.

## Import Policy Contract

- This release MUST enforce allowlist-only import policy.
- Allowed stdlib packages are:
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
- OS, network, or process-sensitive packages such as `os`, `os/exec`, `net`, and `syscall` MUST be blocked.

## Artifact and Reference Contract

- Every action execution MUST persist one artifact JSON document under run-local storage.
- `node.action.executed.output_ref` MUST be present for both successful and failed action executions.
- `output_ref` MUST resolve to the persisted action artifact for the same `run_id`, `node_id`, `step_id`, and `action_index`.
- Canonical reference format is:
  - `run-artifact://node/<node_id>/step/<step_id>/action-<action_index>.json`
- Canonical storage path mapped from this reference is:
  - `./.sigil/runs/<run_id>/artifacts/node/<node_id>/step/<step_id>/action-<action_index>.json`
- Artifact payload MUST include:
  - run, node, step, and action identity
  - `action_type`
  - `language`
  - execution `status`
  - raw `repl_code`
  - captured stdout and stderr
  - execution duration
  - typed error metadata when failed

## Compile Diagnostics Artifact Contract

- On `repl_execution_compile` failures, action artifacts MAY include structured `error_detail`.
- `error_detail` MUST use:
  - `stage` literal `compile`
  - required `message`
  - optional `line`
  - optional `column`
  - optional `symbol`
  - optional `source_line`
- `node.action.executed` payloads remain unchanged and MUST NOT inline compile diagnostics.

## Error Policy Contract

- Non-fatal REPL execution failures (`compile`, `runtime`, `timeout`, `import_policy`, `code_size`) MUST:
  - set `node.action.executed.status=failed`
  - carry deterministic typed error metadata in event and artifact
  - allow the node to continue to the next decision step
- Fatal infrastructure failures (`session initialization`, `artifact persistence`, internal runtime corruption or panic) MUST fail the run with typed error metadata.

## Typed Error Taxonomy

REPL runtime MUST use machine-readable error codes including:

- `repl_session_init`
- `repl_code_size_exceeded`
- `repl_import_blocked`
- `repl_execution_timeout`
- `repl_execution_compile`
- `repl_execution_runtime`
- `repl_artifact_persist`

## Deferred Contracts

- Out-of-process REPL execution backends
- per-run configurable REPL limits and policies
- fine-grained per-package capability profiles
- REPL telemetry streaming beyond persisted artifacts

## Acceptance Scenarios

### Scenario SCN-0000: Selects embedded Go REPL engine architecture for v1

Given runtime architecture rules for the Go REPL  
When the REPL engine is resolved  
Then embedded in-process Go interpretation is selected.

### Scenario SCN-0001: Creates exactly one persistent REPL session per node

Given an active node with no existing REPL session  
When the first continue action executes  
Then exactly one REPL session is created and associated to that node.

### Scenario SCN-0002: Reuses node REPL state across multiple continue steps

Given an active node with existing REPL session state  
When additional continue actions execute for that node  
Then subsequent actions run in the same node REPL session state.

### Scenario SCN-0003: Closes node REPL session on node completion and run termination

Given active node REPL sessions exist  
When a node completes or the run enters a terminal state  
Then the relevant REPL sessions close.

### Scenario SCN-0004: Executes continuation.repl_code in node-local REPL session

Given a node step returns valid `continuation.repl_code`  
When the continue action executes  
Then the code runs in the current node-local REPL session.

### Scenario SCN-0005: Records failed action and continues next step on non-fatal REPL execution error

Given a continue action hits a non-fatal REPL execution error  
When runtime handles the failure  
Then the action is recorded as failed and the node proceeds to the next decision step.

### Scenario SCN-0006: Enforces 180-second execution timeout per action

Given a continue action exceeds the allowed execution time  
When runtime enforces action timeout  
Then the action fails with typed timeout behavior at `180s`.

### Scenario SCN-0007: Rejects repl_code payloads larger than 65536 bytes

Given `continuation.repl_code` exceeds `65536` bytes  
When runtime validates the action input  
Then the action fails with typed code-size behavior.

### Scenario SCN-0008: Truncates stdout/stderr deterministically at 1048576-byte caps

Given action output exceeds the configured stdout or stderr cap  
When runtime captures output  
Then output is truncated deterministically at `1048576` bytes.

### Scenario SCN-0009: Enforces allowlist-only import policy and rejects blocked imports

Given `continuation.repl_code` imports blocked packages  
When runtime validates imports  
Then execution fails with typed import-policy behavior.

### Scenario SCN-0010: Persists per-action artifact and sets node.action.executed.output_ref

Given an action execution completes or fails  
When runtime persists the action artifact  
Then `node.action.executed.output_ref` points to that artifact.

### Scenario SCN-0011: Fails run on fatal REPL infrastructure errors with typed error metadata

Given a fatal REPL infrastructure error occurs  
When runtime handles the failure  
Then the run fails with typed error metadata.

### Scenario SCN-0012: Persists structured compile diagnostics in failed action artifacts for repl_execution_compile errors

Given a continue action fails with `repl_execution_compile`  
When runtime persists the action artifact  
Then structured compile diagnostics are stored in `error_detail` when parseable.

### Scenario SCN-0013: Preserves node.action.executed payload contract while exposing diagnostics through artifact and feedback only

Given structured compile diagnostics are available  
When runtime exposes diagnostic information  
Then `node.action.executed` payload shape remains unchanged and diagnostics appear only in the artifact and downstream feedback surfaces.
