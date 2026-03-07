# PRD-0016: Sigil Deterministic Runtime Governance Guardrails Specification

## Status

Draft

## Context

`sigil` currently supports iterative node execution but requires explicit
harness-level stop contracts for deterministic operational bounds.

This PRD defines deterministic runtime governance guardrails for step-count,
wall-clock, and consecutive-failure thresholds, with typed terminal failure
metadata on policy breaches.

## Goals

- Define v1 harness runtime guardrail keys under run config.
- Define deterministic defaults and validation rules.
- Define deterministic breach terminalization behavior.
- Define typed failure metadata in `run.failed` payloads for guardrail breaches.
- Define recursive/non-recursive parity for guardrail enforcement.

## Non-Goals

- Token accounting guardrails in this phase.
- Cost accounting guardrails in this phase.
- New CLI flags or CLI configuration interfaces for guardrails.

## Run Config Guardrail Contract

Run config adds:

```yaml
guardrails:
  max_steps_per_node: <int>
  max_total_steps_per_run: <int>
  max_run_duration_ms: <int>
  max_consecutive_step_failures: <int>
```

Default values are:

- `guardrails.max_steps_per_node = 64`
- `guardrails.max_total_steps_per_run = 256`
- `guardrails.max_run_duration_ms = 1200000`
- `guardrails.max_consecutive_step_failures = 6`

Validation rules:

- all guardrail values MUST be integers `>= 1`

## Enforcement Contract

Harness MUST enforce guardrails deterministically during node execution:

- `max_steps_per_node`: enforced before appending `node.step.started`
- `max_total_steps_per_run`: enforced before appending `node.step.started`
- `max_run_duration_ms`: enforced as a run-scoped wall-clock deadline across
  active inference and REPL execution, with loop-boundary checks preserving
  deterministic failure metadata
- `max_consecutive_step_failures`: run-scoped counter over consecutive failed
  continue actions (`node.action.executed.status=failed`)

Counter behavior:

- consecutive failure counter increments only on failed continue actions
- counter resets on successful continue action
- counter resets on final decision step

## Terminal Failure Contract

On guardrail breach:

- harness returns typed error code `harness_limit_exceeded`
- run terminalizes as `run.failed`
- payload MUST include deterministic guardrail metadata:
  - `limit_key`
  - `configured_value`
  - `observed_value`
- payload MUST include `failed_node_id`
- payload MAY include `failed_step_id` when a step identifier is available

## Runtime Event Contract Extension

`run.failed` payload adds optional guardrail fields:

- `limit_key` (non-empty when present)
- `configured_value` (non-empty when present)
- `observed_value` (non-empty when present)
- `failed_step_id` (UUIDv7 when present)

Invariant:

- if `limit_key` is present, `configured_value` and `observed_value` MUST both
  be present and non-empty

## Deferred Contract

Token and cost guardrail enforcement is deferred even though accounting-ledger
capture and rollups are delivered in this phase:

- `max_total_tokens`
- `max_total_cost_usd`

## Acceptance Scenarios

### Scenario SCN-0000: Defines run guardrails config section for deterministic step time and failure thresholds

Given run config contract definitions are loaded  
When run guardrail configuration keys are inspected  
Then deterministic guardrail keys for steps time and failure thresholds are defined.

### Scenario SCN-0001: Applies default deterministic guardrail values when guardrails config is omitted

Given run configuration omits `guardrails` section  
When defaults are applied  
Then deterministic default guardrail values are used.

### Scenario SCN-0002: Rejects non-positive deterministic guardrail configuration values

Given run configuration includes non-positive guardrail values  
When run configuration validation runs  
Then initialization fails with validation error.

### Scenario SCN-0003: Enforces max_steps_per_node before appending node.step.started

Given an active node has reached configured `max_steps_per_node`  
When harness attempts to start another step  
Then run fails with typed guardrail breach metadata and no additional step is started for that node.

### Scenario SCN-0004: Enforces max_total_steps_per_run across root and recursive nodes

Given run-scoped total started steps reaches configured `max_total_steps_per_run`  
When harness attempts to start another step in any node  
Then run fails with typed guardrail breach metadata.

### Scenario SCN-0005: Enforces max_run_duration_ms as hard run wall-clock budget

Given an active inference call or continue action exceeds configured
`max_run_duration_ms`  
When run-scoped wall-clock deadline enforcement executes  
Then the active step is interrupted before completion and run fails with typed
guardrail breach metadata.

### Scenario SCN-0006: Enforces max_consecutive_step_failures using consecutive failed continue actions

Given consecutive failed continue actions reach configured threshold  
When harness processes failure accounting  
Then run fails with typed guardrail breach metadata.

### Scenario SCN-0007: Resets consecutive failed-step counter after successful continue action or final decision

Given one or more consecutive failed continue actions were observed  
When a successful continue action or final decision occurs  
Then consecutive failed-step counter resets.

### Scenario SCN-0008: Emits run.failed with harness_limit_exceeded and deterministic limit metadata on breach

Given a deterministic guardrail breach occurs  
When run terminalization is persisted  
Then `run.failed` includes `error_code=harness_limit_exceeded` and deterministic limit metadata.

### Scenario SCN-0009: Includes failed_node_id and optional failed_step_id in run.failed guardrail failures

Given run fails due to deterministic guardrail breach  
When run.failed payload is persisted  
Then payload includes failed_node_id and includes failed_step_id when available.

### Scenario SCN-0010: Applies deterministic guardrails identically in recursive and non-recursive profiles

Given recursive and non-recursive harness profiles  
When deterministic guardrails are enforced  
Then both profiles follow identical guardrail enforcement semantics.

### Scenario SCN-0011: Defers token and cost guardrails pending accounting-ledger subsystem

Given deterministic runtime governance guardrails are defined for this phase  
When guardrail scope is validated  
Then token and cost guardrails remain deferred to accounting-ledger delivery.
