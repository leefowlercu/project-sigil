# PRD-0510: Sigil Accounting Ledger Specification

## Status

Draft

## Context

`sigil` needs a trustworthy accounting surface that can explain the token and
cost impact of model turns, steps, subcalls, nodes, and full runs.

This PRD owns:

- accounting config and fallback pricing
- normalized accounting schemas
- step, node, run, and tree rollup semantics
- canonical accounting exposure in events, artifacts, and CLI output

## Goals

- Define normalized accounting summary and rollup contracts.
- Define gateway-first token and cost accounting semantics.
- Define fallback-pricing behavior when gateways omit cost.
- Define provenance and completeness metadata so mixed-fidelity totals remain auditable.
- Define canonical event, artifact, and CLI exposure for accounting data.
- Define the stable `tree_total` accounting surface consumed by runtime budget guardrails.

## Non-Goals

- defining per-node or per-step accounting budgets
- supporting non-USD currencies in this release
- defining raw provider usage payload storage as part of the canonical event contract

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

- `accounting.pricing_version` defaults to `v1`
- `accounting.fallback_pricing` is optional
- fallback pricing uses the active run `llm.provider` and `llm.model`
- input and output fallback pricing values MUST be positive integers when configured
- `reasoning_microusd_per_million_tokens` is optional and MUST be positive when configured

Representative environment variables:

- `SIGIL_RUN_ACCOUNTING_PRICING_VERSION`
- `SIGIL_RUN_ACCOUNTING_FALLBACK_PRICING_<PROVIDER>_<MODEL>_INPUT_MICROUSD_PER_MILLION_TOKENS`
- `SIGIL_RUN_ACCOUNTING_FALLBACK_PRICING_<PROVIDER>_<MODEL>_OUTPUT_MICROUSD_PER_MILLION_TOKENS`
- `SIGIL_RUN_ACCOUNTING_FALLBACK_PRICING_<PROVIDER>_<MODEL>_REASONING_MICROUSD_PER_MILLION_TOKENS`

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

- `currency` MUST be `usd`
- cost uses integer `microusd`
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
- `complete` MUST require the corresponding total field to be known
- `partial` MUST preserve known subtotals instead of zero-filling missing data
- `unavailable` MUST preserve missing-item counts instead of reporting false zero-complete totals

### `AccountingRollup`

Aggregate scopes MUST expose:

- `model_total`
- `direct_subcalls_total`
- `tree_total`

`tree_total` is the full causal total for the scope, including recursive descendants.

## Inference Accounting Contract

- Gateway-reported token usage MUST be normalized into one accounting summary.
- Gateway-reported cost MUST be preserved at the most granular known level.
- When a gateway reports only total cost, runtime MUST preserve total cost and MUST NOT invent synthetic component splits.
- When a gateway omits cost, runtime MAY derive missing cost fields from `accounting.fallback_pricing`.

Fallback-pricing rules:

- reasoning tokens are a subset of output tokens
- if `reasoning_microusd_per_million_tokens` is omitted, reasoning tokens use the output-token rate
- non-reasoning output tokens are `max(output_tokens - reasoning_tokens, 0)`
- if required token counts are missing, the affected fallback-derived cost fields remain unavailable

## Ledger and Rollup Contract

- The harness MUST maintain an incremental run-scoped ledger.
- Ledger update points are:
  - after every successful model inference
  - after every subcall finishes
  - after every step completes
  - after every node completes
  - when the run reaches completed, failed, or interrupted terminal state
- Aggregates sum only known numeric values.
- If all contributing items are known, aggregate status is `complete`.
- If some contributing items are missing, aggregate status is `partial`.
- If all contributing items are missing, aggregate status is `unavailable`.
- Recursive child-node accounting contributes to ancestor `tree_total` rollups and MUST NOT be double-counted as direct model work.

## Event Artifact and CLI Exposure Contract

Canonical runtime surfaces MUST expose accounting as follows:

- `node.subcall.executed` includes leaf `accounting` and `accounting_ref`
- `node.step.completed` includes step `AccountingRollup` and `accounting_ref`
- `node.completed` includes node `AccountingRollup` and optional `accounting_ref`
- `run.completed`, `run.failed`, and `run.interrupted` include run `AccountingRollup` and optional `accounting_ref`
- `node.turn.model` artifacts include normalized model-turn accounting summaries
- action artifacts include `subcalls[].accounting`
- successful `sigil run start` text summaries include readable run-level accounting rollups
- successful `sigil run start --output json` output includes run-level `accounting`

Canonical accounting artifact locations are:

- `run-output://run/accounting.json`
- `run-output://node/<node_id>/accounting.json`
- `run-output://node/<node_id>/step/<step_id>/accounting.json`
- `run-output://node/<node_id>/step/<step_id>/subcall-<subcall_index>-accounting.json`

Event envelope ownership remains in `PRD-0210`; this PRD owns the accounting
schema and accounting exposure semantics on those runtime surfaces.

## Terminal Behavior Contract

- `run.completed` MUST include final run accounting rollups.
- `run.failed` MUST include partial accounting captured before failure.
- `run.interrupted` MUST include partial accounting captured before interruption.
- Successful CLI output in both text and JSON modes MUST include the same
  run accounting rollup surfaced in `run.completed`.

## Guardrail Integration Contract

- `PRD-0500` owns budget configuration, deterministic enforcement, and
  `run.failed` limit metadata for `max_total_tokens` and `max_total_cost_usd`.
- This PRD owns the accounting ledger semantics those guardrails evaluate.
- `max_total_tokens` consumes `tree_total.total_tokens` and requires
  `tree_total.token_status=complete`.
- `max_total_cost_usd` consumes `tree_total.known_total_cost_microusd` and
  requires `tree_total.cost_status=complete`.
- When an active accounting budget fails closed, `run.failed.accounting`
  remains the full source of truth for partial or unavailable totals and
  provenance.

## Acceptance Scenarios

### Scenario SCN-0000: Defines accounting section in run config schema

Given run config contract definitions are loaded  
When accounting configuration keys are inspected  
Then the `accounting` section is defined.

### Scenario SCN-0001: Applies default accounting pricing version when accounting section is omitted

Given run configuration omits `accounting`  
When defaults are applied  
Then `accounting.pricing_version` resolves to `v1`.

### Scenario SCN-0002: Applies SIGIL_RUN environment overrides for accounting fallback pricing

Given file configuration defines accounting fallback pricing  
And corresponding `SIGIL_RUN_ACCOUNTING_*` environment variables are present  
When run configuration is merged  
Then environment values override file values.

### Scenario SCN-0003: Rejects run configuration when accounting fallback pricing values are non-positive

Given run configuration includes non-positive fallback pricing values  
When validation runs  
Then run configuration is rejected.

### Scenario SCN-0004: Computes fallback cost from configured pricing when gateway omits cost

Given gateway output omits cost  
And configured fallback pricing is available  
When accounting normalization runs  
Then fallback cost is computed from the configured pricing inputs.

### Scenario SCN-0005: Includes accounting rollup in successful run summary and terminal events

Given a completed run captures normalized accounting  
When successful CLI output and terminal events are inspected  
Then run-level accounting rollups are included.

### Scenario SCN-0006: Aggregates recursive subcall tree accounting into node and run totals

Given recursive subcalls execute and produce normalized accounting  
When node and run accounting rollups are inspected  
Then recursive child accounting contributes to ancestor `tree_total` rollups.

### Scenario SCN-0007: Persists subcall leaf accounting in events and action artifacts

Given subcalls execute within a continue action  
When runtime persists observability data  
Then leaf accounting summaries are present in `node.subcall.executed` and action artifact `subcalls[]`.

### Scenario SCN-0008: Preserves partial accounting status when cost is unavailable

Given accounting data omits one or more cost totals  
When normalization and rollup complete  
Then accounting preserves `cost_status=partial` or `cost_status=unavailable` instead of reporting false complete totals.

### Scenario SCN-0009: Includes partial accounting in failed terminal events

Given a run fails after some accounting has been captured  
When `run.failed` is persisted  
Then partial accounting is included.

### Scenario SCN-0010: Includes partial accounting in interrupted terminal events

Given a run is interrupted after some accounting has been captured  
When `run.interrupted` is persisted  
Then partial accounting is included.
