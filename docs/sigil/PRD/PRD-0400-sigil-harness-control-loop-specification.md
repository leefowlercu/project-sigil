# PRD-0400: Sigil Harness Control Loop Specification

## Status

Draft

## Context

`sigil` executes recursive language-model control steps through a harness that
coordinates inference, REPL execution, and recursive child-node orchestration.

This PRD owns the harness control loop:

- root-node startup
- node-local step semantics
- recursive and non-recursive execution profiles
- high-level continuation handling
- high-level child-node orchestration
- run completion and harness-level failure propagation

Prompt composition is defined in `PRD-0320`. Model-input envelopes are defined
in `PRD-0420`. REPL runtime behavior is defined in `PRD-0430`. Subcall API
behavior is defined in `PRD-0440`. Guardrail behavior is defined in `PRD-0500`.

## Goals

- Define the v1 harness control loop for root and recursive nodes.
- Define harness execution profiles selected by `rlm.enabled`.
- Define high-level continuation handling and completion behavior.
- Define how recursive subcalls integrate with the harness lifecycle.

## Non-Goals

- Defining gateway transport behavior.
- Defining detailed REPL session, artifact, or import-policy behavior.
- Defining prompt text content or remote prompt registry behavior.
- Defining app-server query or projection APIs.

## Canonical Terminology

- `step`: one node-local decision cycle
- `turn`: one transcript contribution linked to a step
- `action`: one executable instruction emitted by a `continue` decision
- `root node`: the node created at depth `0` when run execution begins
- `child node`: a node created through recursive harness execution

## Harness Control Loop Contract

- Harness execution MUST begin with exactly one root node at depth `0`.
- Each active node MUST run a control loop that requests structured inference with `schema_id=sigil.rlm.response.v1`.
- Node steps MUST interpret `validated_payload.decision` as follows:
  - `continue`: execute continuation behavior for that node
  - `final`: complete that node with `validated_payload.final.answer`
- Root-node `final` MUST terminate the run as successful completion.

## Execution Profile Contract

- Harness MUST support both execution profiles selected by `rlm.enabled`:
  - `true`: recursive profile where recursive subcalls may create child nodes
  - `false`: non-recursive profile where multi-step continuation remains allowed but child-node creation is disabled
- Recursive and non-recursive profiles MUST preserve the same step, turn, and action semantics.

## Continuation Contract

- `decision=continue` MUST carry exactly one executable action in `continuation.repl_code`.
- Harness MUST execute exactly one continuation action per `continue` step.
- If the continuation payload is structurally invalid, harness progression MUST fail with typed output-validation behavior.
- Detailed REPL runtime behavior is defined in `PRD-0430`.

## Recursive Subcall Contract

- Recursive subcalls execute through the REPL bindings defined in `PRD-0440`.
- When recursion depth permits, recursive subcalls MUST execute as child-node inference.
- Child node depth MUST be `parent_depth + 1`.
- In recursive profile, depth-limit overflow MUST fall back to plain subcall behavior rather than creating a child node.
- In non-recursive profile, recursive subcall APIs remain bound but MUST return typed depth-limit behavior for all invocations.
- Successful child-node `final` output MUST be returned to the caller REPL context.

## Completion and Failure Contract

- Run completion MUST require root-node `decision=final` with non-empty `final.answer`.
- Final-answer evidence validation follows `PRD-0310`.
- Unrecoverable inference, REPL, or harness orchestration errors MUST terminate the run as `failed` with typed error metadata.
- Child-node failures MUST propagate deterministic failure information to caller node context unless escalation policy terminates the run.
- Deterministic runtime governance guardrails MUST be enforced as defined in `PRD-0500`.

## Deferred Contracts

- Dynamic prompt registries sourced from files or remote stores
- fine-grained REPL sandboxing and resource quotas
- multi-REPL language support beyond Go
- asynchronous harness scheduling modes

## Acceptance Scenarios

### Scenario SCN-0000: Starts harness execution with one root node at depth zero

Given valid run configuration and execution inputs  
When harness execution begins  
Then exactly one root node is created at depth `0`.

### Scenario SCN-0001: Executes single continuation repl_code action from structured response in node-local REPL state

Given a node step returns `decision=continue` with valid `continuation.repl_code`  
When harness handles the continue decision  
Then exactly one continuation action executes in node-local REPL state.

### Scenario SCN-0002: Propagates typed output-validation error when continue payload omits continuation repl_code

Given a node step returns `decision=continue` without required `continuation.repl_code`  
When harness validates the structured response  
Then harness fails the step with typed output-validation behavior.

### Scenario SCN-0003: Creates child node through rlm_query only when resulting depth does not exceed rlm.max_depth

Given an active parent node and configured `rlm.max_depth`  
When recursive subcall execution is requested  
Then a child node is created only if `parent_depth + 1 <= rlm.max_depth`.

### Scenario SCN-0004: Falls back to plain llm_query behavior when rlm_query call would exceed rlm.max_depth in recursive mode

Given active parent node depth equals `rlm.max_depth` in recursive profile  
When recursive subcall execution is requested  
Then no child node is created and plain subcall behavior is returned.

### Scenario SCN-0005: Returns child final answer to caller REPL context on successful recursive subcall

Given recursive child-node execution completes successfully  
When the child node reaches `decision=final`  
Then the child `final.answer` is returned to the caller REPL context.

### Scenario SCN-0006: Completes run when root node emits decision final with non-empty final answer

Given the root node returns `decision=final` with non-empty `final.answer`  
When harness processes the terminal decision  
Then the run completes successfully.

### Scenario SCN-0007: Defines step as one node-local decision cycle

Given harness terminology is evaluated  
When one node-local decision cycle is described  
Then that cycle is represented as one `step`.

### Scenario SCN-0008: Records turns as user or model transcript contributions linked to a step

Given a node step executes  
When transcript contributions are persisted  
Then turns are recorded as `user` or `model` contributions linked to that step.

### Scenario SCN-0009: Limits continue steps to exactly one executable action

Given a node step returns `decision=continue`  
When harness handles the decision  
Then exactly one executable action is allowed for that step.

### Scenario SCN-0010: Runs non-recursive multi-step profile when rlm.enabled is false and returns typed depth-limit feedback for recursive subcalls

Given `rlm.enabled=false`  
When harness executes multiple steps and a recursive subcall is attempted  
Then multi-step execution continues in the active node and recursive subcalls return typed depth-limit behavior.

### Scenario SCN-0011: Allows multiple subcalls inside one continuation action while preserving one action per continue step

Given a continuation action issues multiple subcalls  
When harness processes the step  
Then multiple subcalls are allowed while the step still records exactly one action execution.

### Scenario SCN-0012: Applies deterministic runtime guardrails from PRD-0500 during harness execution

Given harness execution is active  
When deterministic runtime guardrails are evaluated  
Then enforcement and breach terminalization follow `PRD-0500`.
