# PRD-0100: Sigil-Web Root and Index Route Specification

## Status

Draft

## Context

`sigil-web` keeps the root and index route family intentionally small. The
application entrypoint exists to hand operators into the primary `/agents`
workflow without maintaining a second home-screen contract.

## Goals

- Define the minimal behavior for the `/` route family.
- Keep the root route aligned with the routed shell described in the ADRs.

## Non-Goals

- Defining application-shell layout behavior beyond the root-route handoff;
  `PRD-0150` owns that contract.
- Defining `/agents` route behavior in detail.
- Defining `/runs/$runId` route behavior in detail.

## Route Contract

- The root path `/` MUST redirect operators to `/agents`.
- The root and index route family MUST remain free of a second operator-facing
  workspace unless a future PRD explicitly adds one.

## Acceptance Scenarios

### Scenario SCN-0000: Redirects the root route to the agents hub

Given the operator opens sigil-web at `/`  
When the application resolves the root route  
Then the application redirects the operator to `/agents`.
