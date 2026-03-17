# ADR

This directory stores architecture decision records for the `sigil-web` subproject.

- Use ADRs to capture long-lived technical decisions and tradeoffs.
- Cross-reference related PRDs and submodule changes when behavior is affected.
- Keep design-governance decisions here when they define how Paper, acceptance,
  and implementation must stay aligned.

## Current ADRs

- [ADR-0001 Sigil-Web Paper Design Governance](ADR-0001-sigil-web-paper-design-governance.md):
  A routed scenario manifest plus a Paper-only design manifest define how
  routed scenarios map to one or more verification lanes.
- [ADR-0002 Sigil-Web TanStack Start Route and State Architecture](ADR-0002-sigil-web-tanstack-start-route-and-state-architecture.md):
  TanStack Start route shell and session-state architecture for the operator
  UI.
- [ADR-0003 Sigil-Web Generated App-Server Client and Acceptance Lanes](ADR-0003-sigil-web-generated-app-server-client-and-acceptance-lanes.md):
  Generated TypeScript client bindings plus fake-server and real-server
  acceptance lanes.

## Next ADR

Create the next record as:

- `docs/sigil-web/ADR/ADR-0004-<slug>.md`
