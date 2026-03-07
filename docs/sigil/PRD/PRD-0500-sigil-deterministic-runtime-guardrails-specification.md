# PRD-0500: Sigil Deterministic Runtime Guardrails Specification

## Status

Draft

## Context

`sigil` needs explicit harness-level stop contracts for deterministic
operational bounds. Those bounds span configuration, enforcement, and terminal
failure metadata, so they need one dedicated normative owner.

This PRD owns:

- run-config guardrail keys and defaults
- guardrail environment overrides
- deterministic enforcement rules
- guardrail-specific `run.failed` metadata

## Goals

- Define deterministic runtime guardrail keys under run config.
- Define defaults and validation rules.
- Define run-scoped token and cost budgets on cumulative run accounting totals.
- Define deterministic breach terminalization behavior.
- Define typed guardrail metadata in `run.failed` payloads.
- Define recursive and non-recursive parity for enforcement.

## Non-Goals

- per-node or per-step accounting budgets
- new CLI flags or alternate configuration interfaces for guardrails
- adaptive or heuristic runtime budgets
- non-USD cost budget inputs

## Run Config Guardrail Contract

Run config adds:

```yaml
guardrails:
  max_steps_per_node: <int>
  max_total_steps_per_run: <int>
  max_run_duration_ms: <int>
  max_consecutive_step_failures: <int>
  max_total_tokens: <int, optional>
  max_total_cost_usd: "<decimal, optional>"
```

Default values are:

- `guardrails.max_steps_per_node = 64`
- `guardrails.max_total_steps_per_run = 256`
- `guardrails.max_run_duration_ms = 1200000`
- `guardrails.max_consecutive_step_failures = 6`
- `guardrails.max_total_tokens` omitted and disabled by default
- `guardrails.max_total_cost_usd` omitted and disabled by default

Validation rules:

- `max_steps_per_node`, `max_total_steps_per_run`, `max_run_duration_ms`, and `max_consecutive_step_failures` MUST be integers `>= 1`
- `max_total_tokens` MUST be an integer `> 0` when configured
- `max_total_cost_usd` MUST be a trimmed base-10 USD decimal string `> 0` with at most 6 fractional digits and no exponent syntax
- accepted `max_total_cost_usd` values MUST be canonicalized before storage and comparison

Representative environment variables:

- `SIGIL_RUN_GUARDRAILS_MAX_STEPS_PER_NODE`
- `SIGIL_RUN_GUARDRAILS_MAX_TOTAL_STEPS_PER_RUN`
- `SIGIL_RUN_GUARDRAILS_MAX_RUN_DURATION_MS`
- `SIGIL_RUN_GUARDRAILS_MAX_CONSECUTIVE_STEP_FAILURES`
- `SIGIL_RUN_GUARDRAILS_MAX_TOTAL_TOKENS`
- `SIGIL_RUN_GUARDRAILS_MAX_TOTAL_COST_USD`

## Accounting Budget Contract

- `max_total_tokens` and `max_total_cost_usd` are run-scoped only and are disabled when omitted.
- Accounting budgets evaluate cumulative run `tree_total` accounting only.
- `max_total_tokens` compares against `tree_total.total_tokens`.
- `max_total_cost_usd` compares against `tree_total.known_total_cost_microusd`.
- `max_total_cost_usd` is parsed exactly from user-facing USD text into internal `microusd`.
- Under an active `max_total_tokens` budget, any `tree_total.token_status != complete` is terminal.
- Under an active `max_total_cost_usd` budget, any `tree_total.cost_status != complete` is terminal.

## Enforcement Contract

Harness MUST enforce guardrails deterministically during node execution:

- `max_steps_per_node`: enforced before appending `node.step.started`
- `max_total_steps_per_run`: enforced before appending `node.step.started`
- `max_run_duration_ms`: enforced as a run-scoped wall-clock deadline across active inference and REPL execution, with loop-boundary checks preserving deterministic failure metadata
- `max_consecutive_step_failures`: run-scoped counter over consecutive failed continue actions where `node.action.executed.status=failed`
- `max_total_tokens`: enforced immediately after each successful `RecordModelTurn` and immediately after each persisted subcall record using run `tree_total` accounting
- `max_total_cost_usd`: enforced immediately after each successful `RecordModelTurn` and immediately after each persisted subcall record using run `tree_total` accounting

Counter behavior:

- the counter increments only on failed continue actions
- the counter resets on successful continue action
- the counter resets on final decision step

Accounting-budget behavior:

- budget evaluation occurs after the causative model turn or subcall event and artifact have been persisted so the triggering sample remains auditable
- active accounting budgets fail closed when required totals are partial or unavailable
- recursive and non-recursive profiles use identical run-scoped `tree_total` accounting semantics

## Terminal Failure Contract

On guardrail breach:

- harness returns typed error code `harness_limit_exceeded`
- the run terminalizes as `run.failed`
- payload MUST include:
  - `limit_key`
  - `configured_value`
  - `observed_value`
  - `failed_node_id`
- payload MAY include `failed_step_id` when a step identifier is available

Accounting-budget metadata rules:

- `configured_value` stores the configured integer for `max_total_tokens`
- `configured_value` stores the canonical USD string for `max_total_cost_usd`
- `observed_value` stores the known cumulative observed total when present, otherwise `unavailable`
- `run.failed.accounting` remains the full source of truth for partial or unavailable accounting details

## Runtime Event Contract Extension

`run.failed` payload adds optional guardrail fields:

- `limit_key`
- `configured_value`
- `observed_value`
- `failed_step_id`

Invariants:

- if `limit_key` is present, `configured_value` and `observed_value` MUST both be present and non-empty
- if `failed_step_id` is present, it MUST be UUIDv7

Event envelope and payload ownership remains in `PRD-0210`; this PRD owns the
guardrail-specific semantics of those fields.

## Acceptance Scenarios

### Scenario SCN-0000: Defines run guardrails config section for deterministic step time failure and accounting budget thresholds

Given run config contract definitions are loaded  
When run guardrail configuration keys are inspected  
Then deterministic guardrail keys for step, time, failure, and accounting budget thresholds are defined.

### Scenario SCN-0001: Applies default deterministic guardrail values when guardrails config is omitted

Given run configuration omits `guardrails`  
When defaults are applied  
Then deterministic default guardrail values are used and accounting budget guardrails remain unset.

### Scenario SCN-0002: Applies SIGIL_RUN environment overrides for deterministic guardrail fields

Given file configuration defines guardrail values  
And corresponding `SIGIL_RUN_GUARDRAILS_*` environment variables are present  
When run configuration is merged  
Then environment values override file values including optional accounting budgets.

### Scenario SCN-0003: Rejects invalid deterministic guardrail configuration values

Given run configuration includes non-positive or malformed guardrail values
When validation runs
Then run configuration is rejected.

### Scenario SCN-0004: Enforces max_steps_per_node before appending node.step.started

Given a node has reached the configured `max_steps_per_node`  
When harness attempts to append the next `node.step.started`  
Then execution is blocked by deterministic guardrail enforcement.

### Scenario SCN-0005: Enforces max_total_steps_per_run across root and recursive nodes

Given cumulative run step count has reached `max_total_steps_per_run`  
When harness attempts to start another step anywhere in the run tree  
Then execution is blocked by deterministic guardrail enforcement.

### Scenario SCN-0006: Enforces max_run_duration_ms as hard run wall-clock budget

Given run wall-clock duration reaches `max_run_duration_ms`  
When harness checks the run-scoped deadline  
Then the run fails with deterministic guardrail behavior.

### Scenario SCN-0007: Enforces max_consecutive_step_failures using consecutive failed continue actions

Given consecutive failed continue actions reach the configured threshold  
When harness evaluates the failure counter  
Then the run fails with deterministic guardrail behavior.

### Scenario SCN-0008: Resets consecutive failed-step counter after successful continue action or final decision

Given the consecutive failed-step counter is non-zero  
When a continue action succeeds or the node reaches a final decision  
Then the counter resets.

### Scenario SCN-0009: Emits run.failed with harness_limit_exceeded and deterministic limit metadata on breach

Given a deterministic guardrail breach occurs  
When the terminal failure payload is persisted  
Then `run.failed` includes `error_code=harness_limit_exceeded` and deterministic limit metadata.

### Scenario SCN-0010: Includes failed_node_id and optional failed_step_id in run.failed guardrail failures

Given a run fails due to deterministic guardrail breach  
When `run.failed` payload is persisted  
Then `failed_node_id` is present and `failed_step_id` is included when available.

### Scenario SCN-0011: Requires configured_value and observed_value when run.failed limit_key is present

Given a `run.failed` payload includes `limit_key`  
When payload validation runs  
Then `configured_value` and `observed_value` are both required.

### Scenario SCN-0012: Validates failed_step_id as UUIDv7 when present in run.failed payload

Given a `run.failed` payload includes `failed_step_id`  
When payload validation runs  
Then `failed_step_id` must be UUIDv7.

### Scenario SCN-0013: Applies deterministic guardrails identically in recursive and non-recursive profiles

Given recursive and non-recursive harness profiles  
When deterministic guardrails are enforced  
Then both profiles follow identical enforcement semantics.

### Scenario SCN-0014: Enforces max_total_tokens on cumulative run accounting tree totals

Given cumulative run accounting tree totals exceed `max_total_tokens`
When deterministic guardrails evaluate the updated run ledger
Then the run fails with deterministic guardrail behavior.

### Scenario SCN-0015: Enforces max_total_cost_usd on cumulative run accounting tree totals

Given cumulative run accounting tree totals exceed `max_total_cost_usd`
When deterministic guardrails evaluate the updated run ledger
Then the run fails with deterministic guardrail behavior.

### Scenario SCN-0016: Fails closed when max_total_tokens sees incomplete tree-total accounting

Given `max_total_tokens` is active
And cumulative run accounting tree totals are incomplete for tokens
When deterministic guardrails evaluate the updated run ledger
Then the run fails with deterministic guardrail behavior.

### Scenario SCN-0017: Fails closed when max_total_cost_usd sees incomplete tree-total accounting

Given `max_total_cost_usd` is active
And cumulative run accounting tree totals are incomplete for cost
When deterministic guardrails evaluate the updated run ledger
Then the run fails with deterministic guardrail behavior.
