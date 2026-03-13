# PRD-0440: Sigil REPL Subcall APIs and Batching Specification

## Status

Draft

## Context

`sigil` exposes plain and recursive subcall APIs inside the node-local Go REPL
so continuation code can delegate work without collapsing all execution into the
root node.

This PRD owns:

- REPL subcall API surface
- plain versus recursive subcall execution modes
- batched subcall behavior
- recursive depth-limit behavior
- small-context local-only recursive fallback behavior
- subcall observability and failure behavior

Inference adapter behavior is defined in `PRD-0300`. Plain-subcall and harness
response schemas are defined in `PRD-0310`. REPL runtime session behavior is
defined in `PRD-0430`.

## Goals

- Define the REPL subcall API surface for plain and recursive workflows.
- Define deterministic mixed batch execution behavior.
- Define recursive depth-limit and non-recursive profile behavior.
- Define deterministic small-context fallback behavior after recursive exploration has already occurred in the same node.
- Define subcall observability contracts in events and action artifacts.
- Define timeout and cancellation behavior for recursive subcalls.

## Non-Goals

- Introducing `llm.recursive_model` in this release
- parallel recursive batching in this release
- user-configurable batch parallelism in this release
- redefining action artifact ownership outside subcall traces

## REPL Subcall API Contract

The node-local Go REPL runtime MUST expose:

- `llm_query(prompt string, context string) (string, error)`
- `rlm_query(prompt string, context string) (string, error)`
- `llm_query_batched(calls []map[string]string) ([]map[string]string, error)`
- `rlm_query_batched(calls []map[string]string) ([]map[string]string, error)`
- `read_action_artifact(action_ref string) (ActionOutput, error)`

Batched call-item input keys MUST be:

- `prompt`
- `context`

Batched result-item keys MUST be:

- `answer`
- `error_code`
- `error_message`

`ActionOutput` MUST expose:

- `Status`
- `Stdout`
- `Stderr`
- `ErrorCode`
- `ErrorMessage`

## Subcall Execution Contract

- `llm_query` MUST execute one plain one-shot inference and MUST NOT create a child node.
- `rlm_query` MUST execute child-node inference when recursion depth permits.
- Successful child-node `final` output MUST be returned to the caller REPL context.
- `llm_query_batched` MUST execute bounded-parallel fan-out and preserve input order in output results.
- `rlm_query_batched` MUST execute sequentially and preserve input order in output results.
- `read_action_artifact` MUST resolve exact action output fields from a canonical current-run `action_ref`.
- After a small-context node has already executed recursive subcalls in a prior continue step, subsequent `rlm_query` and `rlm_query_batched` invocations in that same node MUST fall back to plain subcall behavior instead of creating additional child nodes.
- Recursive APIs (`rlm_query`, `rlm_query_batched`) MUST execute each subcall with an independent `300s` timeout budget derived from run-scoped context, decoupled from parent action elapsed timeout and ancestor recursive subcall deadline depletion across levels.
- Recursive subcalls MUST still honor run-context cancellation.

## Routing Contract

- Subcalls MUST route through the active node run-config provider and model values:
  - `llm.provider`
  - `llm.model`
- No separate recursive model routing key is introduced in this release.

## Depth-Limit Contract

### Recursive profile (`rlm.enabled=true`)

- `rlm_query` at depth limit MUST fall back to plain `llm_query` behavior.
- `rlm_query_batched` at depth limit MUST fall back per item to plain subcall behavior.

### Non-recursive profile (`rlm.enabled=false`)

- Recursive APIs remain bound.
- Recursive APIs MUST return typed `repl_child_depth_limit` behavior for all invocations.
- No child nodes are created.

## Observability Contract

- Every subcall item execution MUST emit `node.subcall.executed` with strict payload invariants.
- Every continue-action artifact MUST persist `subcalls[]` traces with stable per-action subcall indexing.
- Continue-step contract remains unchanged: exactly one `continuation.repl_code` action per `decision=continue` step.

## Schema and Fallback Contract

- Plain subcalls MUST resolve responses through strict schema `sigil.llm.answer.v1`.
- Raw-text fallback policy is owned by `PRD-0300` and applies only to `sigil.llm.answer.v1`.

## Failure Contract

- Plain or batched subcall item failures SHOULD surface as structured subcall result errors without immediate run failure.
- Subcall event persistence failures are infrastructure failures and MUST fail the run with typed infrastructure metadata.

## Deferred Contracts

- Per-call model override fields in subcall APIs
- extended batched payload schemas beyond `prompt` and `context`
- richer recursive scheduling policies beyond sequential recursive batching
- widening `read_action_artifact` beyond the narrow `ActionOutput` surface

## Acceptance Scenarios

### Scenario SCN-0000: Exposes llm_query rlm_query llm_query_batched and rlm_query_batched in node-local REPL session

Given an initialized node-local Go REPL session  
When REPL bindings are inspected  
Then `llm_query`, `llm_query_batched`, `rlm_query`, and `rlm_query_batched` are available.

### Scenario SCN-0001: Executes llm_query as plain one-shot inference without child node creation

Given an active node-local REPL session  
When `llm_query` is invoked  
Then one plain subcall inference executes and no child node is created.

### Scenario SCN-0002: Creates child node for rlm_query when depth is within limit

Given an active node-local REPL session and recursion depth permits child creation  
When `rlm_query` is invoked  
Then a child node is created and executed.

### Scenario SCN-0003: Returns child final answer to caller REPL context on successful subcall

Given a recursive child node completes successfully  
When `rlm_query` returns  
Then the child final answer is returned to the caller REPL context.

### Scenario SCN-0004: Falls back to llm_query when rlm_query reaches max depth in recursive mode

Given recursive execution is at maximum configured depth  
When `rlm_query` is invoked  
Then plain subcall behavior is used instead of creating a child node.

### Scenario SCN-0005: Executes llm_query_batched with bounded parallel fan-out and preserves input-order results

Given an active node-local REPL session with multiple independent plain subcall items  
When `llm_query_batched` is invoked  
Then bounded parallel fan-out executes and result ordering matches input ordering.

### Scenario SCN-0006: Executes rlm_query_batched sequentially and preserves input-order results

Given an active node-local REPL session with multiple recursive subcall items  
When `rlm_query_batched` is invoked  
Then subcall items execute sequentially and result ordering matches input ordering.

### Scenario SCN-0007: Returns structured per-item batched results with answer and typed error metadata

Given batched subcall item execution includes mixed success and failure outcomes  
When batched results are returned  
Then each result item includes `answer`, `error_code`, and `error_message`.

### Scenario SCN-0008: Routes subcalls through active node llm provider and model configuration

Given an active node has configured `llm.provider` and `llm.model`  
When a subcall executes  
Then the subcall routes through that provider and model configuration.

### Scenario SCN-0009: Preserves typed depth-limit behavior for recursive subcalls in non-recursive profile

Given `rlm.enabled=false`  
When `rlm_query` or `rlm_query_batched` is invoked  
Then typed depth-limit behavior is returned and no child nodes are created.

### Scenario SCN-0010: Emits node.subcall.executed event for each subcall item with strict payload invariants

Given one or more subcall items execute inside a continue action  
When observability is persisted  
Then each executed subcall item emits `node.subcall.executed` with strict payload invariants.

### Scenario SCN-0011: Persists subcall traces in action artifacts with stable subcall indexing

Given one or more subcalls execute inside a continue action  
When the action artifact is persisted  
Then subcall traces are stored with stable per-action subcall indexing.

### Scenario SCN-0012: Resolves plain-subcall responses through strict schema sigil.llm.answer.v1

Given a plain subcall executes successfully  
When response validation runs  
Then the response is validated against strict schema `sigil.llm.answer.v1`.

### Scenario SCN-0013: Fails run on subcall event persistence failures with typed infrastructure metadata

Given subcall execution succeeds but subcall event persistence fails  
When the failure is handled  
Then the run fails with typed infrastructure metadata.

### Scenario SCN-0014: Surfaces plain-subcall inference failures as structured subcall result errors without immediate run failure

Given a plain or batched subcall item hits inference failure  
When the subcall returns to caller context  
Then the failure is surfaced as structured subcall result error without immediate run failure.

### Scenario SCN-0015: Applies independent 300-second recursive timeout budget across recursive subcall levels for rlm_query and rlm_query_batched invocations

Given recursive subcalls execute through `rlm_query` or `rlm_query_batched`  
When timeout budgets are evaluated  
Then each recursive subcall receives an independent `300s` timeout budget decoupled from parent action elapsed time.

### Scenario SCN-0016: Cancels recursive subcalls on run-context cancellation despite timeout decoupling from parent action context

Given a recursive subcall is active  
When run-context cancellation occurs  
Then the recursive subcall is canceled even though its timeout budget is decoupled from parent action elapsed time.

### Scenario SCN-0017: Falls back to llm_query after a small-context node already used recursive subcalls in a prior step

Given a small-context node has already executed recursive subcalls in a prior continue step  
When `rlm_query` is invoked in a subsequent step of that same node  
Then plain subcall behavior is used instead of creating another child node.

### Scenario SCN-0018: Exposes read_action_artifact helper in node-local REPL session

Given an initialized node-local Go REPL session  
When REPL bindings are inspected  
Then `read_action_artifact` is available.
