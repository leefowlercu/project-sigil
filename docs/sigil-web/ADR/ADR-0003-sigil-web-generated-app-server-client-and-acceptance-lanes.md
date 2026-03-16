# ADR-0003: Sigil-Web Generated App-Server Client and Acceptance Lanes

## Status

Accepted

## Date

2026-03-15

## Context

`sigil-web` should not hand-maintain a second copy of the Sigil app-server
protocol. The `sigil` submodule already owns typed protocol definitions and
deterministic generation commands for TypeScript and JSON Schema. The web UI
also needs acceptance coverage that is both fast during UI work and trustworthy
against the real server contract.

## Decision

Sigil-web adopts these contract and testing rules:

- TypeScript app-server request, response, and notification types MUST be
  generated from `sigil app-server generate-ts`.
- Browser acceptance MUST remain Gherkin-titled and route through Playwright.
- Acceptance coverage MUST run in two lanes:
  - a fake or scriptable WebSocket server lane for deterministic UI-state and
    error-state coverage
  - a real `sigil app-server serve` lane for contract confidence against the
    actual server surface

## Decision Details

- Generated client artifacts SHOULD live under a dedicated `src/lib/appserver`
  boundary once implementation begins.
- The fake-server lane SHOULD exercise edge states that are awkward or slow to
  provoke through the real server, including protocol incompatibility, heartbeat
  loss, reconnect timing, and malformed input errors.
- The real-server lane SHOULD exercise at least:
  - initialize and `initialized`
  - `run/list`
  - `run/read`
  - `run/subscribe`
  - `run/start`
  - `run/stop`
- Acceptance scenario titles remain the behavioral source of truth even when
  the fake-server lane is used.

## Alternatives Considered

- Handwrite WebSocket models in `sigil-web`: rejected because protocol drift
  would be likely and hard to detect.
- Use only mocked browser tests: rejected because the real app-server contract
  is central to the product.
- Use only real-server end-to-end tests: rejected because UI iteration would be
  slower and harder to make deterministic.

## Consequences

### Positive

- Wire contracts stay coupled to the typed server source of truth.
- UI work can move quickly without giving up real contract checks.
- The acceptance suite can cover both behavioral breadth and backend fidelity.

### Negative

- The web UI now depends on an explicit generation step from the `sigil`
  submodule.
- Two acceptance lanes require fixture discipline to keep expectations aligned.

## Related Documents

- [Sigil-Web ADR Index](README.md)
- [ADR-0001 Paper Design Governance](ADR-0001-sigil-web-paper-design-governance.md)
- [ADR-0002 TanStack Start Route and State Architecture](ADR-0002-sigil-web-tanstack-start-route-and-state-architecture.md)
- [PRD-0100 Session and Connection State](../PRD/PRD-0100-sigil-web-session-and-connection-state-specification.md)
- [PRD-0400 Live Orchestration and Connection Recovery](../PRD/PRD-0400-sigil-web-live-orchestration-and-connection-recovery-specification.md)
- [PRD-0500 Run Control and Authoring](../PRD/PRD-0500-sigil-web-run-control-and-authoring-specification.md)
