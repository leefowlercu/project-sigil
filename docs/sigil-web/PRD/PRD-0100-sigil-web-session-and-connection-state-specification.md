# PRD-0100: Sigil-Web Session and Connection State Specification

## Status

Draft

## Context

`sigil-web` needs a stable session model for talking to the Sigil app-server in
WebSocket mode. Operators must be able to tell whether the server is ready,
incompatible, or unhealthy before attempting orchestration actions.

## Goals

- Define the session bootstrap flow for the web UI.
- Define compatible, incompatible, degraded, and reconnecting connection
  states.
- Preserve operator context when the session degrades and reconnects.

## Non-Goals

- Defining run list, run detail, or run-control layout in detail.
- Defining authentication, TLS, or non-localhost deployment policy.

## Session Contract

- The UI MUST connect to a configured app-server WebSocket endpoint.
- The UI MUST send `initialize` with supported protocol versions and client
  identity before issuing operational requests.
- The UI MUST send `initialized` after a successful initialize response before
  treating the session as ready.
- The ready state MUST display server identity sufficient for operator context,
  including instance and negotiated protocol metadata.
- Incompatible protocol negotiation MUST block the main workspace until the
  operator updates configuration or the server surface.
- Missed heartbeat detection MUST move the UI into a degraded state and trigger
  reconnect behavior.
- Reconnect behavior MUST preserve the intended route and active run selection
  so the workspace can resume in context.

## Error and Recovery Contract

- Incompatible-session UI MUST surface the machine-actionable domain error code
  returned by the server.
- Reconnecting UI MUST distinguish between initial disconnected state and a
  previously healthy session that is attempting recovery.
- Session health UI MUST be available globally so route-level workflows do not
  invent their own connection-state vocabulary.

## Acceptance Scenarios

### Scenario SCN-0000: Connects to a compatible app-server and enters ready operator state

Given the operator configures a reachable Sigil app-server WebSocket endpoint  
When the UI completes `initialize` and `initialized` successfully  
Then the application enters a ready state with operator-visible server identity
and protocol details.

### Scenario SCN-0001: Blocks the workspace with an incompatible-server state when protocol negotiation fails

Given the operator connects to an app-server that does not support the UI's
protocol versions  
When `initialize` fails with an incompatible version error  
Then the application blocks the main workspace and shows the incompatible
server state plus machine-actionable error code.

### Scenario SCN-0002: Marks the session degraded and begins reconnecting after a missed heartbeat

Given the application previously reached a ready connected state  
When the idle heartbeat window is missed  
Then the application marks the session degraded and begins reconnecting without
discarding the operator's current workspace intent.
