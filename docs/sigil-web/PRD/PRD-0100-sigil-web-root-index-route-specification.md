# PRD-0100: Sigil-Web Root and Index Route Specification

## Status

Draft

## Context

`sigil-web` now treats the root and index route family as the primary operator
workspace. The root path owns the agent workspace and selected-agent
deep-linking contract directly instead of redirecting operators into a second
route family.

## Goals

- Define the primary operator-visible behavior for the `/` route family.
- Preserve shareable selected-agent route state at the root route.
- Keep the root route aligned with the routed shell described in the ADRs.

## Non-Goals

- Defining application-shell layout behavior beyond root-route workspace
  ownership;
  `PRD-0150` owns that contract.
- Defining `/runs/$runId` route behavior in detail.

## Route Contract

- The root path `/` MUST render the primary agent workspace without redirecting
  operators to another route.
- The root and index route family MUST inherit the application-shell layout
  contract defined in `PRD-0150`.
- The root and index route family MUST support selected-agent deep-linking
  through the `agent` search parameter.
- Opening `/?agent=<agent-id>` MUST preserve the selected-agent intent in a
  refresh-stable and shareable route shape.

## Acceptance Scenarios

### Scenario SCN-0000: Renders the primary agent workspace at the root route

Given the operator opens sigil-web at `/`  
When the application resolves the root route  
Then the application renders the primary agent workspace without redirecting
the operator to another route.

### Scenario SCN-0001: Deep-links the selected agent in the root route

Given the application can resolve a valid agent identity  
When the operator opens `/?agent=<agent-id>`  
Then the application preserves that selected-agent intent in the `/` route
state.
