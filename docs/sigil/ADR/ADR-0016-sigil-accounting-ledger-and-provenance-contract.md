# ADR-0016: Sigil Accounting Ledger and Provenance Contract

## Status

Accepted

## Context

`sigil` already normalizes model outputs, persists durable runtime events, and
supports recursive subcall execution. It does not yet expose a trustworthy
token-and-cost accounting surface for completed runs, intermediate steps, or
recursive subcall trees.

Without a first-class accounting contract, operators cannot distinguish between
gateway-reported totals, fallback-priced totals, and unknown costs. That makes
run summaries hard to trust and blocks future guardrail enforcement on token or
cost budgets.

## Decision

For v1 accounting delivery, `sigil` adds an incremental runtime ledger with the
following contract:

- Accounting is USD-only and stores cost as integer `microusd`.
- Gateway/provider-reported token and cost values are preferred when available.
- When gateways omit cost, configured fallback pricing may derive cost from
  normalized tokens.
- Missing token or cost values are never coerced to zero.
- Every accounting payload carries provenance and completeness fields:
  - `token_source`
  - `token_status`
  - `cost_source`
  - `cost_status`
  - `pricing_key`
  - `pricing_version`
  - `missing_token_item_count`
  - `missing_cost_item_count`
- Accounting is emitted both inline on canonical runtime events and via
  canonical accounting artifacts for subcall, step, node, and run scopes.
- Aggregate scopes use a fixed rollup model:
  - `model_total`
  - `direct_subcalls_total`
  - `tree_total`
- Recursive accounting contributes to ancestor `tree_total` rollups without
  being double-counted as direct model work.

## Deferred

The accounting ledger is delivered now, but guardrail enforcement based on that
ledger remains deferred:

- `max_total_tokens`
- `max_total_cost_usd`

## Consequences

### Positive

- Run, node, step, and subcall totals become auditable.
- Partial and unavailable costs remain machine-detectable instead of appearing
  falsely exact.
- Future token and cost guardrails can enforce against a stable accounting
  contract without redesigning runtime events or artifacts.

### Negative

- Runtime payloads and artifacts become larger because accounting is persisted
  at multiple scopes.
- Mixed-fidelity accounting requires explicit provenance handling throughout
  inference, harness, and validation paths.

## References

- `PRD-0120: run config core contract`
- `PRD-0210: run event envelope and durability contract`
- `PRD-0410: run start command execution contract`
- `PRD-0440: REPL subcall observability contract`
- `PRD-0500: deterministic runtime guardrails specification`
- `PRD-0510: accounting ledger specification`
