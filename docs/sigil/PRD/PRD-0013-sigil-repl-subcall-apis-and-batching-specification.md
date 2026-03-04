# PRD-0013: Sigil REPL Subcall APIs and Batching Specification

## Status

Draft

## Context

`PRD-0009` and `PRD-0010` define core harness and Go REPL runtime behavior.
Priority 1 introduces first-class cheap subcalls and batched subcalls so
context-heavy retrieval can execute efficiently without forcing recursive child
node orchestration for simple tasks.

## Goals

- Define REPL subcall API expansion for plain and batched workflows.
- Define deterministic mixed batch execution behavior.
- Define recursive max-depth fallback semantics for recursive profile.
- Define non-recursive profile compatibility behavior.
- Define subcall observability contracts in events and action artifacts.

## Non-Goals

- Introducing `llm.recursive_model` in v1.
- Parallel recursive batching in v1.
- User-configurable batch parallelism in v1.

## REPL Subcall API Contract

The node-local Go REPL runtime MUST expose:

- `llm_query(prompt string, context string) (string, error)`
- `rlm_query(prompt string, context string) (string, error)`
- `llm_query_batched(calls []map[string]string) ([]map[string]string, error)`
- `rlm_query_batched(calls []map[string]string) ([]map[string]string, error)`

Batched call-item input keys MUST be:

- `prompt`
- `context`

Batched result-item keys MUST be:

- `answer`
- `error_code`
- `error_message`

## Subcall Execution Contract

- `llm_query` MUST execute one plain one-shot inference and MUST NOT create a
  child node.
- `llm_query_batched` MUST execute bounded-parallel fan-out and preserve input
  order in output results.
- `rlm_query_batched` MUST execute sequentially and preserve input order in
  output results.

## Routing Contract

- Subcalls MUST route through active node run-config provider/model values:
  - `llm.provider`
  - `llm.model`
- No separate recursive model routing key is introduced in v1.

## Depth-Limit Contract

### Recursive profile (`rlm.enabled=true`)

- `rlm_query` at depth limit MUST fallback to plain `llm_query` behavior.
- `rlm_query_batched` at depth limit MUST fallback per item to plain subcall
  behavior (`llm_query`/`llm_query_batched` path).

### Non-recursive profile (`rlm.enabled=false`)

- Recursive APIs remain bound.
- Recursive APIs MUST return typed `repl_child_depth_limit` behavior for all
  invocations.
- No child nodes are created.

## Observability Contract

- Every subcall item execution MUST emit `node.subcall.executed` with strict
  payload invariants.
- Every continue-action artifact MUST persist `subcalls[]` traces with stable
  per-action subcall indexing.
- Continue-step contract remains unchanged: exactly one
  `continuation.repl_code` action per `decision=continue` step.

## Plain Subcall Schema Contract

- Plain one-shot subcalls MUST resolve and validate against strict schema:
  - `sigil.llm.answer.v1`
- `sigil.llm.answer.v1` MUST require:
  - `answer` (string, non-empty)

## Failure Contract

- Subcall item failures in plain/batched execution SHOULD be surfaced as
  structured subcall result errors without immediate run failure.
- Subcall event persistence failures are infrastructure failures and MUST fail
  run with typed infrastructure metadata.

## Deferred Contracts

- Per-call model override fields in subcall APIs.
- Extended batched payload schemas beyond `prompt`/`context`.

## Acceptance Scenarios

### Scenario SCN-0000: Exposes llm_query llm_query_batched and rlm_query_batched in node-local Go REPL

Given an initialized node-local Go REPL session  
When REPL bindings are inspected  
Then llm_query llm_query_batched and rlm_query_batched are available.

### Scenario SCN-0001: Executes llm_query as plain one-shot inference without child node creation

Given an active node-local REPL session  
When llm_query is invoked  
Then one plain subcall inference executes and no child node is created.

### Scenario SCN-0002: Executes llm_query_batched with bounded parallel fan-out and preserves input-order results

Given an active node-local REPL session with multiple independent subcall items  
When llm_query_batched is invoked  
Then bounded parallel fan-out executes and result ordering matches input ordering.

### Scenario SCN-0003: Executes rlm_query_batched sequentially and preserves input-order results

Given an active node-local REPL session with multiple recursive subcall items  
When rlm_query_batched is invoked  
Then subcall items execute sequentially and result ordering matches input ordering.

### Scenario SCN-0004: Returns structured per-item batched results with answer and typed error metadata

Given batched subcall item execution includes mixed success and failure outcomes  
When batched result payload is returned  
Then each item includes answer error_code and error_message fields.

### Scenario SCN-0005: Routes subcalls through active node llm provider and model configuration

Given run config defines active llm provider and model  
When subcalls execute from node-local REPL  
Then subcalls use active node llm provider and model routing values.

### Scenario SCN-0006: Falls back to llm_query when rlm_query reaches max depth in recursive mode

Given recursive profile is enabled and parent depth is at recursion limit  
When rlm_query is invoked  
Then recursive child creation is skipped and plain llm_query fallback behavior executes.

### Scenario SCN-0007: Falls back to llm_query_batched item execution when rlm_query_batched reaches max depth in recursive mode

Given recursive profile is enabled and batched recursive subcall items reach recursion limit  
When rlm_query_batched is invoked  
Then recursive child creation is skipped per item and plain subcall fallback executes.

### Scenario SCN-0008: Preserves typed depth-limit behavior for recursive subcalls in non-recursive profile

Given non-recursive profile is active  
When rlm_query or rlm_query_batched is invoked  
Then typed depth-limit behavior is returned and no child nodes are created.

### Scenario SCN-0009: Emits node.subcall.executed event for each subcall item with strict payload invariants

Given one or more subcall items execute inside a continue action  
When runtime events are persisted  
Then one node.subcall.executed event is emitted per subcall item with strict payload invariants.

### Scenario SCN-0010: Persists subcall traces in action artifacts with stable subcall indexing

Given a continue action executes subcall items  
When action artifact persistence runs  
Then action artifact includes stable-indexed subcalls trace records.

### Scenario SCN-0011: Allows multiple subcalls inside one continuation action while preserving one action per continue step

Given a continue step executes repl_code that performs multiple subcalls  
When step execution completes  
Then exactly one continuation action is recorded and multiple subcalls are observed inside that action.

### Scenario SCN-0012: Resolves plain-subcall responses through strict schema sigil.llm.answer.v1

Given a plain subcall inference request executes  
When schema resolution and validation run  
Then response is validated against strict schema sigil.llm.answer.v1.

### Scenario SCN-0013: Fails run on subcall event persistence failures with typed infrastructure metadata

Given subcall event persistence fails during continue action execution  
When failure is propagated  
Then run transitions to failed with typed infrastructure metadata.

### Scenario SCN-0014: Surfaces plain-subcall inference failures as structured subcall result errors without immediate run failure

Given a plain subcall inference attempt fails for one subcall item  
When subcall result is returned to REPL caller  
Then failure is surfaced as structured subcall result error and run does not fail immediately.
