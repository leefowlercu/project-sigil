# ADR-0015: Sigil Deterministic Runtime Governance Guardrails

## Status

Accepted

## Context

The harness node loop is intentionally iterative (`continue` decisions can
repeat), but there is no first-class deterministic stop policy for run-level and
node-level execution budgets.

Without explicit guardrails, run behavior is less predictable, more expensive,
and harder to diagnose when pathological loops or repeated failures occur.

## Decision

For v1, `sigil` adds deterministic runtime governance guardrails enforced by the
harness:

- `max_steps_per_node`
- `max_total_steps_per_run`
- `max_run_duration_ms`
- `max_consecutive_step_failures`

These are configured via run config under `guardrails.*` and include defaults.

When any guardrail is breached, harness execution terminalizes the run as
`failed` with typed metadata:

- `error_code=harness_limit_exceeded`
- deterministic limit metadata (`limit_key`, `configured_value`,
  `observed_value`)
- `failed_node_id`
- optional `failed_step_id`

Guardrails are enforced identically in recursive and non-recursive execution
profiles.

## Deferred

Token and cost guardrails are explicitly deferred in this phase:

- `max_total_tokens`
- `max_total_cost_usd`

They will be added in a later accounting-ledger subsystem delivery.

## Consequences

### Positive

- Deterministic, bounded harness execution.
- Better failure diagnostics for policy-stop terminalization.
- Stronger operational safety in long-running recursive workloads.

### Negative

- Additional runtime accounting complexity in harness execution paths.
- Existing runs that relied on effectively unbounded loops now fail earlier
  under explicit policy.

## References

- `PRD-0500: deterministic runtime guardrails specification`
- `PRD-0210: run.failed payload contract`
- `PRD-0400: harness control-loop contract`
