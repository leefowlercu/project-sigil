# PRD-0410: Sigil Run Start Command Execution Specification

## Status

Draft

## Context

`PRD-0130` defines input-path behavior for `sigil run start`, while runtime
execution semantics were previously deferred.

`sigil` now has lifecycle, event, inference, harness, and Go REPL contracts.
This PRD specifies how `sigil run start` executes those contracts as a blocking
runtime command.

## Goals

- Define `sigil run start` as a blocking harness execution entrypoint.
- Define template rendering behavior for `prompt_template` and
  `context_template` using `--var key=value`.
- Define recursive and non-recursive execution profile behavior selected by
  `rlm.enabled`.
- Define default human-readable progress and summary output for successful runs.
- Define JSON compatibility output for machine-consumer workflows.
- Define fatal error propagation and non-zero exit behavior.

## Non-Goals

- Defining asynchronous job queue semantics for run execution.
- Defining app-server run-control APIs.
- Defining streaming inference behavior in v1.
- Defining `sigil run stop` behavior owned by
  `PRD-0450-sigil-run-stop-command-execution-specification.md`.

## Command Runtime Contract

- `sigil run start` MUST execute synchronously and block until run terminal
  state.
- Command MUST initialize lifecycle with queued source `cli.run.start` and
  resolved config-path metadata.
- Command MUST create exactly one root node before first inference step.
- Harness step processing MUST follow contracts in `PRD-0400` and `PRD-0430`.
- REPL subcall API behavior and observability MUST follow `PRD-0440`.

## Prompt and Context Resolution Contract

- Effective prompt source MUST be exactly one of:
  - `prompt`
  - rendered `prompt_template`
- Effective context source MUST be exactly one of:
  - `context`
  - rendered `context_template`
- Effective context initializes node-local REPL scope and model-input metadata.
- Harness MUST NOT replay full effective context body into outbound model-step
  inference messages.
- Template rendering engine MUST be Go `text/template` with
  `missingkey=error`.
- `--var key=value` MAY be provided multiple times.
- Duplicate template keys MUST resolve deterministically using last value.
- Invalid `--var` entries MUST fail command validation.

## Execution Profile Contract

### Recursive Profile (`rlm.enabled=true`)

- Harness MUST allow child node creation via `rlm_query` when depth does not
  exceed `rlm.max_depth`.
- Depth-limit overflows MUST fallback to plain-subcall behavior for recursive
  APIs.

### Non-Recursive Profile (`rlm.enabled=false`)

- Harness MUST continue multi-step execution in the active node.
- Harness MUST NOT create child nodes.
- REPL `rlm_query` binding MUST remain available.
- Every `rlm_query` call MUST return typed `repl_child_depth_limit` error.

## Step and Event Contract

For each step, harness MUST emit canonical events and maintain strict ordering:

1. `node.step.started`
2. `node.turn.user`
3. `node.turn.model`
4. zero or more `node.subcall.executed` (continue steps only when subcalls occur)
5. `node.action.executed` (continue steps only)
6. `node.step.completed`

## Completion and Output Contract

- Root `decision=final` with non-empty answer and resolvable `final.evidence[]`
  refs MUST complete run successfully.
- Run-level accounting exposure semantics MUST follow `PRD-0510`.

### Default Text Output Contract

- When `--output` is omitted or set to `text`, `sigil run start` MUST render an
  append-only human-readable stdout stream.
- Text-mode output MUST begin with a preflight summary including:
  - resolved application config path
  - resolved run config path
  - selected gateway
  - selected provider/model
  - execution profile
  - `rlm.max_depth`
- Text-mode live progress MUST be driven by persisted canonical lifecycle events
  and MUST NOT depend on logger strings.
- Text-mode live progress MUST surface:
  - `run.queued` and `run.running` as run-state lines with full `run_id`
  - `node.started` as indented root or child node lines using node depth and role
  - `node.step.started` and `node.step.completed` as step-progress lines
  - `node.subcall.executed` as architecture lines showing recursive, fallback,
    or plain execution mode plus duration and child-node linkage when present
  - `node.action.executed` as continue-action completion or failure lines
  - `node.completed` and `node.failed` as node-terminal lines
  - terminal run events as the trigger for the final summary
- Successful text-mode terminal summary MUST include:
  - `state`
  - `run_id`
  - duration
  - `events_path`
  - `final_answer_ref`
  - full final answer text
  - readable run-level accounting rollups
  - accounting artifact reference and path when present

### JSON Compatibility Output Contract

- When `--output json` is requested, successful command output MUST be one JSON
  object including:
  - `run_id`
  - `state`
  - `final_answer`
  - `final_answer_ref`
  - `accounting`
  - `events_path`
- The v1 JSON field set and field names MUST remain backward compatible with the
  existing successful `sigil run start` result payload.

## Failure Contract

- Unrecoverable harness, inference, template-render, or runtime infrastructure
  errors MUST terminate run as `failed` with typed error metadata.
- Deterministic runtime governance guardrail breaches MUST terminate run as
  `failed` with typed `harness_limit_exceeded` metadata and deterministic limit
  fields.
- Command MUST exit non-zero on terminal failed run.

## Deferred Contracts

The following are deferred to future PRDs:

- Asynchronous run execution control.
- User-configurable template engines.

## Acceptance Scenarios

### Scenario SCN-0000: Executes sigil run start as a blocking harness entrypoint

Given valid application and run configuration inputs  
When a user runs `sigil run start`  
Then command blocks until the run reaches a terminal state.

### Scenario SCN-0001: Emits run.queued with source cli.run.start and resolved config-path metadata

Given command runtime initialization begins  
When lifecycle is created for a CLI invocation  
Then `run.queued` is persisted with `source=cli.run.start` and resolved config
path metadata.

### Scenario SCN-0002: Starts lifecycle and root node before first inference step

Given a run transitions from queued to running  
When harness execution starts  
Then exactly one root node exists before first inference step execution.

### Scenario SCN-0003: Resolves effective prompt and context from direct fields or rendered templates

Given run configuration with valid prompt/context direct fields or templates  
When effective inference inputs are resolved  
Then harness uses the resolved effective prompt and context for REPL
initialization and model-input metadata.

### Scenario SCN-0004: Fails run start on template rendering errors under strict missing-key policy

Given run configuration template fields with missing required template variables  
When strict template rendering executes  
Then command fails with typed template render error behavior.

### Scenario SCN-0005: Executes multi-step decision loop until root terminal decision

Given harness execution is active for the root node  
When model responses continue returning step decisions  
Then harness continues step execution until root emits terminal final decision.

### Scenario SCN-0006: Persists per-step user and model turns and emits node turn events

Given a node step executes  
When transcript contributions are persisted  
Then `node.turn.user` and `node.turn.model` events are emitted for the step.

### Scenario SCN-0007: Executes exactly one continue action per continue decision and emits node.action.executed

Given model returns `decision=continue` with one continuation action  
When harness handles continue decision  
Then exactly one action executes and one `node.action.executed` event is
persisted.

### Scenario SCN-0008: Allows recursive child-node execution via rlm_query when recursion is enabled and depth permits

Given `rlm.enabled=true` and active parent node depth within limits  
When REPL code calls `rlm_query`  
Then harness creates and executes a child node.

### Scenario SCN-0009: Falls back to plain subcall behavior when recursion exceeds rlm.max_depth in recursive mode

Given recursive execution at maximum configured depth  
When REPL code calls `rlm_query`  
Then no child node is created and plain-subcall fallback behavior is returned.

### Scenario SCN-0010: Runs non-recursive multi-step profile when rlm.enabled is false

Given `rlm.enabled=false` and active harness execution  
When model emits continue decisions across multiple steps  
Then harness executes multiple steps without creating child nodes.

### Scenario SCN-0011: Returns typed depth-limit error for all rlm_query calls in non-recursive mode

Given `rlm.enabled=false` with an active node-local REPL session  
When REPL code calls `rlm_query`  
Then typed depth-limit error is returned for the call.

### Scenario SCN-0012: Completes run on root final answer and sets run.completed.final_answer_ref

Given root node emits `decision=final` with non-empty answer  
And final evidence references resolve successfully  
When harness finalization executes  
Then run completes and `run.completed.final_answer_ref` is set.

### Scenario SCN-0013: Prints human-readable run progress and terminal summary by default

Given run reaches completed terminal state  
When command returns success  
Then stdout contains human-readable preflight, progress, and terminal summary
lines including `state`, `run_id`, `events_path`, `final_answer_ref`,
`final_answer`, and `accounting`.

### Scenario SCN-0014: Exits non-zero with typed failure metadata on unrecoverable harness inference or template errors

Given unrecoverable harness inference template or runtime infrastructure failure  
When command handles terminal failure  
Then command exits non-zero and failure metadata is typed and deterministic.

### Scenario SCN-0015: Exits non-zero when deterministic runtime guardrail breach terminalizes run

Given deterministic runtime guardrail breach occurs during harness execution  
When command handles terminal failed run  
Then command exits non-zero and run.failed contains deterministic guardrail breach metadata.

### Scenario SCN-0016: Prints JSON run summary on successful completion when output json is requested

Given run reaches completed terminal state  
When command returns success with `--output json`  
Then stdout contains the compatibility JSON summary with `run_id`, `state`,
`final_answer_ref`, `events_path`, `final_answer`, and `accounting`.

### Scenario SCN-0017: Shows recursive child-node growth in text output

Given recursive child-node execution occurs during `sigil run start`  
When text-mode progress is rendered  
Then stdout includes indented child-node lines and recursive subcall
architecture progress.

### Scenario SCN-0018: Shows fallback subcall mode in text output when recursion depth is exceeded

Given recursive APIs exceed configured `rlm.max_depth` during `sigil run start`  
When text-mode progress is rendered  
Then stdout includes fallback subcall progress showing the fallback execution
mode.
