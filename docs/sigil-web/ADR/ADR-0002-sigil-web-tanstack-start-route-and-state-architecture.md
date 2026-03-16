# ADR-0002: Sigil-Web TanStack Start Route and State Architecture

## Status

Accepted

## Date

2026-03-15

## Context

`sigil-web` will be a command, control, and orchestration plane for the Sigil
app-server running over WebSocket. The UI needs one route structure that can:

- bootstrap and monitor the app-server session
- list and inspect runs
- subscribe to live run updates
- start and stop runs
- preserve operator context across refreshes and reconnects

## Decision

Sigil-web adopts TanStack Start with these architecture rules:

- one application shell route that owns connection status and primary
  navigation
- one session store responsible for WebSocket lifecycle, handshake state,
  heartbeat health, and reconnect intent
- route-level workflow boundaries aligned to the design manifest:
  - `/connect`
  - `/runs`
  - `/runs/$runId`
  - `/runs/new`
- route-state identifiers in the design manifest are the stable names for the
  operator-visible UI states implemented by these routes

## Decision Details

- Session state such as `ready`, `incompatible`, `degraded`, and `reconnecting`
  is application-wide and MUST not be reimplemented independently in individual
  pages.
- Run detail routes MUST compose read-plane calls (`run/read`, `run/tree/read`,
  `run/steps/list`, `run/step/read`, `run/artifact/read`) over one shared
  client/service layer rather than embedding ad hoc protocol calls in route
  components.
- Live subscriptions MUST layer onto the same route state instead of switching
  to a second live-only page model.
- The route structure SHOULD preserve the current run context during refresh and
  reconnect workflows so operators do not lose place while observing active
  runs.

## Alternatives Considered

- Use a flat single-page shell with custom in-memory routing: rejected because
  route state would become hard to trace back to PRDs and artboards.
- Put connection state inside each page independently: rejected because session
  health is global and must stay consistent across the app.
- Split static read screens and live screens into separate route families:
  rejected because it would duplicate operator workflows around the same run.

## Consequences

### Positive

- Route boundaries stay aligned with PRD ownership and Paper artboards.
- Session and reconnect behavior can be implemented once and reused everywhere.
- Run inspection and live orchestration share one workspace mental model.

### Negative

- The shell and session store become foundational infrastructure that must exist
  before deeper route work feels complete.
- Route-state naming must stay disciplined because it is reused across docs,
  tests, and design artifacts.

## Related Documents

- [Sigil-Web ADR Index](README.md)
- [ADR-0001 Paper Design Governance](ADR-0001-sigil-web-paper-design-governance.md)
- [ADR-0003 Generated App-Server Client and Acceptance Lanes](ADR-0003-sigil-web-generated-app-server-client-and-acceptance-lanes.md)
- [PRD-0100 Session and Connection State](../PRD/PRD-0100-sigil-web-session-and-connection-state-specification.md)
- [PRD-0200 Operator Shell and Run List](../PRD/PRD-0200-sigil-web-operator-shell-and-run-list-specification.md)
- [PRD-0300 Run Detail Workspace](../PRD/PRD-0300-sigil-web-run-detail-workspace-specification.md)
- [PRD-0400 Live Orchestration and Connection Recovery](../PRD/PRD-0400-sigil-web-live-orchestration-and-connection-recovery-specification.md)
- [PRD-0500 Run Control and Authoring](../PRD/PRD-0500-sigil-web-run-control-and-authoring-specification.md)
