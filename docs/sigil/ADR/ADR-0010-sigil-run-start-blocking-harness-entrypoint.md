# ADR-0010: Sigil Run Start Blocking Harness Entrypoint

## Status

Accepted

## Date

2026-03-03

## Context

`sigil` now defines lifecycle, event, inference, harness, and REPL runtime contracts,
but `sigil run start` still behaves as a configuration bootstrap command.

To enable deterministic end-to-end testing and delivery of the v1 harness behavior,
`sigil run start` must become the synchronous runtime execution entrypoint.

## Decision

`sigil run start` MUST execute the harness loop synchronously (blocking) until the
run reaches a terminal state.

Execution profile selection in v1 is controlled by `run_config.rlm.enabled`:

- `true`: recursive harness execution with child nodes enabled up to `rlm.max_depth`.
- `false`: non-recursive multi-step execution profile where child node creation is
  disabled.

## Decision Details

- `sigil run start` MUST initialize config/run-config and immediately start harness
  execution in the same invocation.
- Command success output MUST be a machine-readable JSON summary containing at
  minimum: `run_id`, terminal `state`, `final_answer`, `final_answer_ref`, and
  `events_path`.
- Template rendering for `prompt_template` and `context_template` MUST use Go
  `text/template` with strict `missingkey=error` behavior.
- CLI variable injection for templates MUST use repeatable `--var key=value`
  flags with deterministic last-write-wins behavior for duplicate keys.
- In non-recursive mode (`rlm.enabled=false`), `rlm_query` MUST remain bound but
  MUST return typed `repl_child_depth_limit` errors for all invocations.

## Alternatives Considered

- Keep `run start` as config-initialization only: rejected because it blocks
  runtime validation and acceptance testing.
- Make `run start` asynchronous: rejected for v1 because it complicates
  determinism and immediate operator feedback.
- Remove `rlm_query` binding in non-recursive mode: rejected to keep REPL API
  stable across execution profiles.

## Consequences

- `run start` becomes an end-to-end harness execution command suitable for local
  acceptance testing.
- CLI and harness internals now share an explicit orchestration boundary.
- Non-recursive mode is supported without introducing a second command surface.

## Related Documents

- [ADR-0009 Go REPL Engine and Runtime Boundary](ADR-0009-sigil-go-repl-engine-and-runtime-boundary.md)
- [PRD-0410 Run Start Command Execution Specification](../PRD/PRD-0410-sigil-run-start-command-execution-specification.md)
- [PRD-0400 Harness Control Loop Specification](../PRD/PRD-0400-sigil-harness-control-loop-specification.md)
- [PRD-0430 Go REPL Runtime Specification](../PRD/PRD-0430-sigil-go-repl-runtime-specification.md)
