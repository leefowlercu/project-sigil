# PRD-0017: Sigil Accounting Ledger Specification

## Status

Draft

## Context

`sigil` needs a trustworthy accounting surface that can explain the token and
cost impact of:

- one model turn
- one step
- one subcall
- one node
- one full run
- one recursive subcall tree

This PRD defines normalized token and cost accounting behavior across
inference, harness rollups, runtime events, durable artifacts, and successful
CLI run summaries.

## Goals

- Define one normalized accounting summary and rollup contract for v1.
- Define gateway-first token and cost accounting semantics.
- Define fallback-pricing behavior when gateways omit cost.
- Define provenance and completeness metadata so mixed-fidelity totals remain
  auditable.
- Define step, node, run, and recursive-tree rollup semantics.
- Define canonical event and artifact exposure for accounting data.

## Non-Goals

- Enforcing token-budget or cost-budget guardrails in this phase.
- Supporting non-USD currencies in v1.
- Defining raw provider usage payload storage as part of the canonical event
  contract.

## Run Configuration Contract

Run config adds:

```yaml
accounting:
  pricing_version: <string>
  fallback_pricing:
    <provider>:
      <model>:
        input_microusd_per_million_tokens: <int>
        output_microusd_per_million_tokens: <int>
        reasoning_microusd_per_million_tokens: <int>
```

Configuration rules:

- `accounting.pricing_version` defaults to `v1`.
- `accounting.fallback_pricing` is optional.
- Fallback pricing uses the active run `llm.provider` and `llm.model`.
- Input and output fallback pricing values MUST be positive integers when
  configured.
- `reasoning_microusd_per_million_tokens` is optional and MUST be a positive
  integer when configured.

## Normalized Accounting Contract

### `AccountingSummary`

Every leaf and aggregate accounting total MUST expose:

- `currency`
- token counts:
  - `input_tokens`
  - `output_tokens`
  - `total_tokens`
  - `reasoning_tokens`
- known cost fields:
  - `known_input_cost_microusd`
  - `known_output_cost_microusd`
  - `known_reasoning_cost_microusd`
  - `known_total_cost_microusd`
- provenance and completeness metadata:
  - `token_source`
  - `token_status`
  - `cost_source`
  - `cost_status`
  - `pricing_key`
  - `pricing_version`
  - `missing_token_item_count`
  - `missing_cost_item_count`

Invariant rules:

- `currency` MUST be `usd`.
- Cost uses integer `microusd`.
- `token_source` MUST be one of:
  - `gateway_reported`
  - `mixed`
  - `unavailable`
- `cost_source` MUST be one of:
  - `gateway_reported`
  - `fallback_pricing`
  - `mixed`
  - `unavailable`
- `token_status` and `cost_status` MUST be one of:
  - `complete`
  - `partial`
  - `unavailable`
- `complete` MUST require the corresponding total field to be known.
- `partial` MUST preserve known subtotals instead of zero-filling missing data.
- `unavailable` MUST preserve missing-item counts instead of reporting false
  zero-complete totals.

### `AccountingRollup`

Aggregate scopes MUST expose:

- `model_total`
- `direct_subcalls_total`
- `tree_total`

`tree_total` is the full causal total for the scope, including recursive
descendants.

## Inference Accounting Contract

Inference is the accounting ingress.

- Gateway-reported token usage MUST be normalized into one accounting summary.
- Gateway-reported cost MUST be preserved at the most granular known level.
- When a gateway reports only total cost, runtime MUST preserve total cost and
  MUST NOT invent synthetic component splits.
- When a gateway omits cost, runtime MAY derive missing cost fields from
  `accounting.fallback_pricing`.

Fallback-pricing rules:

- Reasoning tokens are a subset of output tokens.
- If `reasoning_microusd_per_million_tokens` is omitted, reasoning tokens use
  the output-token rate.
- Non-reasoning output tokens are `max(output_tokens - reasoning_tokens, 0)`.
- If required token counts are missing, the affected fallback-derived cost
  fields remain unavailable.

## Ledger and Rollup Contract

The harness MUST maintain an incremental run-scoped ledger.

Ledger update points:

- after every successful model inference
- after every subcall finishes
- after every step completes
- after every node completes
- when the run reaches completed failed or interrupted terminal state

Rollup rules:

- Aggregates sum only known numeric values.
- If all contributing items are known, aggregate status is `complete`.
- If some contributing items are missing, aggregate status is `partial`.
- If all contributing items are missing, aggregate status is `unavailable`.
- Recursive child-node accounting contributes to ancestor `tree_total` rollups
  and MUST NOT be double-counted as direct model work.

## Event Artifact and CLI Exposure Contract

Canonical runtime surfaces MUST expose accounting as follows:

- `node.subcall.executed` includes leaf `accounting` and `accounting_ref`
- `node.step.completed` includes step `AccountingRollup` and `accounting_ref`
- `node.completed` includes node `AccountingRollup` and optional
  `accounting_ref`
- `run.completed`, `run.failed`, and `run.interrupted` include run
  `AccountingRollup` and optional `accounting_ref`
- `node.turn.model` artifacts include the normalized model-turn accounting
  summary
- action artifacts include `subcalls[].accounting`
- successful `sigil run start` JSON output includes run-level `accounting`

Canonical accounting artifact locations are:

- `run-output://run/accounting.json`
- `run-output://node/<node_id>/accounting.json`
- `run-output://node/<node_id>/step/<step_id>/accounting.json`
- `run-output://node/<node_id>/step/<step_id>/subcall-<subcall_index>-accounting.json`

## Terminal Behavior Contract

- `run.completed` MUST include final run accounting rollups.
- `run.failed` MUST include partial accounting captured before failure.
- `run.interrupted` MUST include partial accounting captured before
  interruption.
- Successful CLI output MUST include the same run accounting rollup surfaced in
  `run.completed`.

## Deferred Contract

The accounting ledger is the required precursor for later enforcement, but
guardrail enforcement is deferred in this PRD:

- `max_total_tokens`
- `max_total_cost_usd`

## Acceptance Scenarios

### Scenario SCN-0000: Defines accounting section in run config schema

Given run config contract definitions are loaded  
When accounting configuration keys are inspected  
Then the accounting section is defined in the run config schema.

### Scenario SCN-0001: Applies default accounting pricing version when accounting section is omitted

Given merged run configuration omits the accounting section  
When defaults are applied  
Then `accounting.pricing_version` resolves to `v1`.

### Scenario SCN-0002: Applies SIGIL_RUN environment overrides for accounting fallback pricing

Given run configuration values and corresponding accounting environment
variables  
When merge precedence is applied  
Then SIGIL_RUN accounting environment values override file and default values.

### Scenario SCN-0003: Rejects run configuration when accounting fallback pricing values are non-positive

Given merged run configuration includes non-positive accounting fallback-pricing
values  
When validation runs  
Then initialization fails with accounting fallback-pricing validation error.

### Scenario SCN-0004: Computes fallback cost from configured pricing when gateway omits cost

Given normalized token usage is known and gateway cost is unavailable  
When fallback pricing is configured for the active provider and model  
Then fallback-priced cost is computed and surfaced in completed run
accounting.

### Scenario SCN-0005: Includes accounting rollup in successful run summary and terminal events

Given completed harness execution captures normalized accounting  
When successful CLI output and terminal events are inspected  
Then run summary and terminal events include accounting rollups.

### Scenario SCN-0006: Aggregates recursive subcall tree accounting into node and run totals

Given recursive subcalls execute and produce normalized accounting  
When node and run rollups are inspected  
Then recursive child-node accounting is included in ancestor tree totals.

### Scenario SCN-0007: Persists subcall leaf accounting in events and action artifacts

Given one or more subcalls execute inside a continue action  
When event and artifact payloads are inspected  
Then leaf accounting summaries are persisted for subcall events and action
artifacts.

### Scenario SCN-0008: Preserves partial accounting status when cost is unavailable

Given accounting omits one or more cost totals  
When normalized accounting payloads are inspected  
Then partial or unavailable cost status is preserved instead of reporting
zero-complete totals.

### Scenario SCN-0009: Includes partial accounting in failed terminal events

Given partial accounting is captured before a run fails  
When failed terminal events are inspected  
Then `run.failed` includes the partial accounting rollup.

### Scenario SCN-0010: Includes partial accounting in interrupted terminal events

Given partial accounting is captured before a run is interrupted  
When interrupted terminal events are inspected  
Then `run.interrupted` includes the partial accounting rollup.
