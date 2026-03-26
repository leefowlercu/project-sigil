# ADR-0002: Sigil-Web App-Server Session and Data Boundary

## Status

Accepted

## Context

`sigil-web` consumes `sigil` app-server protocol types, establishes browser
sessions over WebSocket, and supports both demo and live data modes.

The spec suite needs a stable boundary between:

- generated wire types from `sigil`
- browser session management and protocol negotiation
- UI-facing view state and demo fixtures

## Decision

- Generated protocol types from `sigil` remain the wire-source of truth.
- `sigil-web` owns client-side protocol negotiation, session lifecycle, and UI
  state assembly on top of those generated types.
- Demo mode and live mode share the same UI contracts, but demo data may satisfy
  those contracts without an active app-server session.
- Live session behavior is acceptance-backed through the browser UI rather than
  through direct protocol-only acceptance tests.

## Consequences

- Protocol version changes must update generated types and the negotiation layer
  before related UI work can be considered complete.
- Acceptance scenarios for live behavior should drive the UI through real browser
  interactions and mocked WebSocket fixtures, not direct DOM-only helpers.

## Related Documents

- [`../PRD/PRD-0300-sigil-web-live-app-server-session-and-run-subscription-specification.md`](../PRD/PRD-0300-sigil-web-live-app-server-session-and-run-subscription-specification.md)
