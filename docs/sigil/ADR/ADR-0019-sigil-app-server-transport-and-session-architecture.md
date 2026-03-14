# ADR-0019: Sigil App-Server Transport and Session Architecture

## Status

Accepted

## Date

2026-03-13

## Context

`sigil` already has the right internal execution shape for an app-server:

- append-only canonical events in `events.jsonl`
- derived read models for list/status/inspect/events
- durable artifact refs for large payloads
- a CLI surface that currently owns read/control entrypoints directly

Adding an app-server should not create a second execution engine or a second
query stack. It needs one transport/session architecture that layers on top of
existing runtime authority and can later grow from local stdio to richer
clients.

## Decision

Sigil app-server adopts these architecture rules:

- one JSON-RPC 2.0 protocol surface
- one handshake flow:
  - `initialize`
  - `initialized`
- one shared read plane below both CLI and app-server surfaces
- stdio JSONL as the first implemented transport
- WebSocket as the remote transport on the same protocol surface
- stable wire identity `serverName="sigil"` with operator-facing
  `instanceId`/`instanceName` supplied separately from application config

## Decision Details

- `events.jsonl` remains the authoritative runtime source of truth.
- CLI `run list/status/inspect/events` and future app-server read methods MUST
  resolve through one shared query package rather than separate implementations.
- Connection-local session state MUST gate method execution:
  - requests before `initialize` are rejected
  - repeated `initialize` is rejected
  - non-handshake requests before `initialized` are rejected
- Transport framing is line-delimited JSON on stdio in the first slice.
- Protocol bindings and JSON Schema bundles MUST be generated from the typed
  protocol definitions rather than hand-maintained copies.
- App-server process behavior belongs in grouped application config under
  `app_server`.

## Alternatives Considered

- Expose app-server behavior directly from Cobra command handlers: rejected
  because it would keep CLI commands as the only reusable entrypoints.
- Design separate HTTP and stdio protocols: rejected because it would multiply
  method contracts and client bindings too early.
- Introduce app-server-specific read logic first and reconcile later: rejected
  because it would create contract drift between CLI and app-server surfaces.

## Consequences

### Positive

- App-server can grow incrementally without changing runtime authority.
- Rich clients get one typed protocol surface and deterministic generated
  artifacts.
- CLI and app-server stay aligned on shared read-model semantics.

### Negative

- Early transport slices require config, verifier, and acceptance plumbing
  before richer app-server behavior feels visible.
- WebSocket shares one protocol surface with stdio, so later subscription and
  remote-hardening work must extend that shared contract instead of creating a
  parallel remote API.

## Related Documents

- [Sigil ADR Index](README.md)
- [ADR-0005 Event-Sourced Run Architecture](ADR-0005-sigil-event-sourced-run-architecture.md)
- [PRD-0100 Application Configuration](../PRD/PRD-0100-sigil-application-config-specification.md)
- [PRD-0160 Run Inspection Command](../PRD/PRD-0160-sigil-run-inspection-command-specification.md)
- [PRD-0170 App-Server Transport and Handshake](../PRD/PRD-0170-sigil-app-server-transport-and-handshake-specification.md)
