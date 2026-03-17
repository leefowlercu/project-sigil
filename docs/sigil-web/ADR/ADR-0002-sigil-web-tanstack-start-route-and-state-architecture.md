# ADR-0002: Sigil-Web TanStack Start Route and State Architecture

## Status

Accepted

## Date

2026-03-15

## Context

`sigil-web` will be a command, control, and orchestration plane for the Sigil
app-server running over WebSocket. The UI needs one route structure that can:

- bootstrap and monitor the app-server session
- manage and inspect connected agents
- list and inspect runs for a selected agent
- subscribe to live run updates
- start and stop runs
- preserve operator context across refreshes and reconnects

## Decision

Sigil-web adopts TanStack Start with these architecture rules:

- one application shell route that owns connection status and primary
  navigation
- one session store responsible for WebSocket lifecycle, handshake state,
  heartbeat health, and reconnect intent
- route-level workflow boundaries aligned to the verification manifest:
  - `/agents`
  - `/runs/$runId`
- `/` MUST redirect to `/agents`
- selected-agent state on `/agents` MUST be deep-linkable with the `agent`
  search parameter
- route-state identifiers in the verification manifest are the stable names for the
  operator-visible UI states implemented by these routes

## Decision Details

- Session state such as `ready`, `incompatible`, `degraded`, and `reconnecting`
  is application-wide and MUST not be reimplemented independently in individual
  pages.
- The `/agents` hub MUST own fleet visibility, selected-agent detail, and
  selected-agent run discovery instead of splitting those concerns across
  separate connection and run-index pages.
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
- [PRD-0100 Root and Index Route](../PRD/PRD-0100-sigil-web-root-index-route-specification.md)
- [PRD-0200 Agents Route](../PRD/PRD-0200-sigil-web-agents-route-specification.md)
- [PRD-0300 Run Detail Route](../PRD/PRD-0300-sigil-web-run-detail-route-specification.md)
