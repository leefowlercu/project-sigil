# PRD-0170: Sigil App-Server Transport and Handshake Specification

## Status

Draft

## Context

`sigil` needs an additive app-server control plane that reuses runtime
authority, shared query logic, and typed contracts instead of exposing CLI-only
behavior as the only integration surface.

The implemented transport slices focus on:

- grouped application config for app-server process behavior
- a new `sigil app-server` CLI surface
- stdio JSON-RPC transport
- WebSocket JSON-RPC transport for localhost/dev clients
- handshake/session gating
- app-level heartbeat notifications for idle remote connections
- ready and live health endpoints on the remote listener
- deterministic TypeScript and JSON Schema generation

## Goals

- Define the `sigil app-server` parent command and transport subcommands.
- Define JSON-RPC 2.0 handshake requirements shared across stdio and WebSocket.
- Define localhost/dev WebSocket listener behavior, origin validation, and
  health endpoints.
- Define stable domain error signaling under `error.data.code`.
- Define deterministic protocol artifact generation from typed source of truth.

## Non-Goals

- Defining SSE or alternate HTTP transport variants beyond the WebSocket
  listener and health endpoints.
- Defining the full run read plane, live execution, or stop behavior.
- Defining auth, TLS, or non-localhost deployment policy.

## Command Surface Contract

The app-server CLI surface MUST be:

```text
sigil app-server
├── serve
├── generate-ts
└── generate-json-schema
```

- `sigil app-server serve` MUST accept:
  - `--listen stdio://`
  - `--listen ws://127.0.0.1:PORT`
- `sigil app-server generate-ts` MUST emit deterministic TypeScript bindings.
- `sigil app-server generate-json-schema` MUST emit deterministic JSON Schema
  bundles.

## Transport Contract

- Stdio transport MUST be JSONL.
- Each inbound stdio line MUST contain one JSON-RPC 2.0 message.
- Each outbound stdio line MUST contain one JSON-RPC 2.0 response.
- WebSocket transport MUST reuse the same JSON-RPC request, response, and
  notification envelopes as stdio.
- Each inbound WebSocket text frame MUST contain one JSON-RPC 2.0 message.
- Each outbound WebSocket text frame MUST contain one JSON-RPC 2.0 message.
- Requests before `initialize` MUST be rejected.
- Repeated `initialize` on one connection MUST be rejected.
- Non-handshake requests after `initialize` but before `initialized` MUST be
  rejected.

## Remote Listener Contract

- WebSocket listener support MUST be localhost/dev only in v1.
- Non-loopback bind targets MUST be rejected.
- Browser-style connections with an `Origin` header MUST be validated against
  `app_server.allowed_origins`.
- `app_server.websocket.path` MUST identify the WebSocket upgrade route.
- `app_server.health.ready_path` and `app_server.health.live_path` MUST return
  success while the listener is serving requests.

## Connection Health Contract

- Idle WebSocket connections MUST receive periodic `server/heartbeat`
  notifications.
- `server/heartbeat` notifications MUST include:
  - `instanceId`
  - `serverTime`
  - `heartbeatIntervalMs`
- Ordinary outbound traffic MAY satisfy liveness between heartbeat emissions,
  but idle connections MUST still receive heartbeats.

## Initialization Contract

- The initialize request MUST carry client info plus one or more supported
  protocol versions.
- The initialize response MUST include:
  - stable `serverName`
  - build-derived `serverVersion`
  - operator-facing `instanceId`
  - operator-facing `instanceName`
  - negotiated `protocolVersion`
  - advertised method families
  - advertised capabilities
- The initialize response MUST advertise `capabilities.config` with default and
  supported config versions for rich clients.

## Error Contract

- Protocol-layer parse, invalid-request, method-not-found, invalid-params, and
  internal failures MUST use standard JSON-RPC error codes.
- Sigil machine-actionable domain codes MUST be carried under
  `error.data.code`.
- Phase-1 stable domain codes include:
  - `handshake_required`
  - `initialized_required`
  - `already_initialized`
  - `unsupported_protocol_version`
  - `method_not_found`
  - `invalid_params`

## Protocol Artifact Generation Contract

- `generate-ts` and `generate-json-schema` MUST be generated from the typed
  protocol definitions used by the server.
- Generated outputs MUST be deterministic so fixture drift can be detected in
  CI.

## Acceptance Scenarios

### Scenario SCN-0000: Exposes app-server subcommands under sigil

Given the `sigil` executable is available  
When a user runs `sigil app-server --help`  
Then the CLI exposes `serve`, `generate-ts`, and `generate-json-schema`.

### Scenario SCN-0001: Rejects app-server requests before initialize on stdio

Given the phase-1 app-server stdio transport is running  
When a client sends a non-handshake request before `initialize`  
Then the server returns a JSON-RPC error with standard invalid-request code and
domain code `handshake_required`.

### Scenario SCN-0002: Rejects repeated initialize on stdio

Given the phase-1 app-server stdio transport is running  
When a client sends `initialize` twice on one connection  
Then the second request fails with standard invalid-request code and domain
code `already_initialized`.

### Scenario SCN-0003: Serves server ping after initialize handshake on stdio

Given the phase-1 app-server stdio transport is running  
When a client completes `initialize` and `initialized`  
Then `server/ping` succeeds and returns Sigil server identity plus negotiated
protocol version.

### Scenario SCN-0004: Generates deterministic TypeScript bindings from typed app-server protocol definitions

Given the app-server protocol definitions are unchanged  
When `sigil app-server generate-ts` runs twice  
Then both generated TypeScript outputs match exactly.

### Scenario SCN-0005: Generates deterministic JSON Schema bundles from typed app-server protocol definitions

Given the app-server protocol definitions are unchanged  
When `sigil app-server generate-json-schema` runs twice  
Then both generated JSON Schema outputs match exactly.

### Scenario SCN-0006: Serves server ping after initialize handshake on WebSocket

Given the app-server WebSocket transport is running on a localhost listener  
When a client connects on the configured WebSocket path and completes
`initialize` and `initialized`  
Then `server/ping` succeeds with the same Sigil identity contract used on
stdio.

### Scenario SCN-0007: Exposes ready and live health endpoints while WebSocket listener is serving

Given the app-server WebSocket transport is running on a localhost listener  
When a client reads the configured ready and live endpoints  
Then both endpoints return success while the listener is serving requests.

### Scenario SCN-0008: Emits heartbeat notifications on idle WebSocket connections

Given the app-server WebSocket transport is running on a localhost listener  
When a client completes the initialize handshake and remains idle  
Then the server emits `server/heartbeat` notifications with stable heartbeat
metadata.
