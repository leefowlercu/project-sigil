# ADR-0012: Sigil REPL Subcall Modes Batching and Observability

## Status

Accepted

## Context

`sigil` currently supports recursive REPL subcalls through `rlm_query(prompt, context)`.
For Priority 1 workloads (large-context chunked retrieval), the runtime needs
cheaper one-shot subcalls and batched subcalls with deterministic observability.

The architecture must preserve the existing one-action-per-continue-step
contract while allowing many subcalls within that action.

## Decision

1. `sigil` v1 REPL API is expanded with first-class subcall functions:
   - `llm_query(prompt string, context string) (string, error)`
   - `llm_query_batched(calls []map[string]string) ([]map[string]string, error)`
   - `rlm_query_batched(calls []map[string]string) ([]map[string]string, error)`
2. `llm_query` executes as plain one-shot inference and does not create child
   nodes.
3. Mixed batch execution profile is selected for v1:
   - `llm_query_batched` executes bounded-parallel with stable input-order
     output.
   - `rlm_query_batched` executes sequentially with stable input-order output.
4. Recursive max-depth behavior in recursive mode is fallback-based:
   - `rlm_query` falls back to plain `llm_query` when recursion depth limit is
     reached.
   - `rlm_query_batched` falls back per-item to plain batched/plain subcall
     execution when recursion depth limit is reached.
5. Non-recursive harness mode (`rlm.enabled=false`) keeps typed depth-limit
   behavior for recursive APIs and does not create child nodes.
6. Subcall routing remains on existing run config only:
   - `llm.provider`
   - `llm.model`
   No `llm.recursive_model` run-config key is introduced in v1.
7. Batched APIs return structured per-item results with keys:
   - `answer`
   - `error_code`
   - `error_message`
8. Subcall observability is mandatory in both runtime events and action
   artifacts:
   - New canonical runtime event: `node.subcall.executed`
   - Action artifacts persist stable-indexed `subcalls[]` traces.

## Consequences

- High-volume retrieval workloads can use cheap plain calls and batched calls
  without forcing child-node orchestration for every subtask.
- Runtime retains deterministic event ordering and one-action-per-step
  semantics while increasing within-action expressiveness.
- Event and artifact observability improve auditability and diagnosis for
  subcall-heavy runs.
- Model-routing behavior remains simple in v1, reducing configuration and
  migration complexity.

## Deferred

- Dedicated `llm.recursive_model` routing controls.
- Parallel recursive batched execution.
- User-configurable batch parallelism caps.
- Compatibility shims for pre-Priority-1 subcall contracts.
