# ADR-0017: Sigil Local Run Stop PID Signal Control

## Status

Accepted

## Date

2026-03-07

## Context

`sigil` v1 runs are local blocking CLI processes started by `sigil run start`.
The runtime already persists durable run events and supports interruption state,
but there is no control transport for stopping an active CLI run.

`sigil run stop` needs a deterministic local control mechanism that:

- works without app-server APIs,
- preserves `events.jsonl` as the run-state source of truth,
- supports graceful interruption instead of abrupt process death,
- remains simple enough for v1 delivery on Unix-like hosts.

## Decision

`sigil run stop` uses local run-directory metadata plus `SIGTERM` as the v1
control transport for gracefully stopping active CLI runs on Unix-like hosts,
with process identity guarded by PID start-time validation.

## Decision Details

- Active CLI runs publish `process.json` under `./.sigil/runs/<run_id>/`
  before `run.queued` becomes externally observable.
- `process.json` records `run_id`, `pid`, `recorded_at`, `started_at`, and
  `source=cli.run.start`.
- Stop commands validate the live PID plus `started_at` identity before sending
  `SIGTERM`, and MUST fail safely when metadata is stale or reused.
- Stop requests publish `stop-request.json` under the same run directory before
  signaling the target process.
- `SIGTERM` is the canonical external stop signal in v1.
- `sigil run start` traps `SIGTERM` and converts it into graceful run
  interruption rather than immediate process exit.
- `events.jsonl` remains the authoritative run-state source; auxiliary control
  files do not become the source of truth.
- Non-Unix transport behavior is deferred.

## Alternatives Considered

- Poll-only stop markers without OS signaling: rejected because stop latency
  becomes dependent on step-loop boundaries and active blocking work.
- App-server control API first: rejected because it expands scope beyond local
  CLI control needs for v1.
- Cross-platform abstract transport in v1: rejected because it would add
  portability complexity before the local contract is proven.

## Consequences

- Local CLI runs gain a deterministic graceful-stop path without requiring a
  separate daemon or server.
- The runtime must maintain auxiliary per-run process metadata and signal-aware
  shutdown handling.
- v1 explicitly scopes this control path to Unix-like hosts.

## Related Documents

- [ADR-0005 Event-Sourced Run Architecture](ADR-0005-sigil-event-sourced-run-architecture.md)
- [ADR-0010 Run Start Blocking Harness Entrypoint](ADR-0010-sigil-run-start-blocking-harness-entrypoint.md)
- [PRD-0450 Run Stop Command Execution Specification](../PRD/PRD-0450-sigil-run-stop-command-execution-specification.md)
- [PRD-0200 Run Lifecycle State Machine Specification](../PRD/PRD-0200-sigil-run-lifecycle-state-machine-specification.md)
- [PRD-0210 Run Event Contract Specification](../PRD/PRD-0210-sigil-run-event-contract-specification.md)
