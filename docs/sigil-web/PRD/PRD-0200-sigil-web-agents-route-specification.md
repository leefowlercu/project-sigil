# PRD-0200: Sigil-Web Agents Route Specification

## Status

Draft

## Context

The `/agents` route is the primary routed workspace in `sigil-web`. For the
current reset, the only preserved operator-visible contract is selected-agent
deep-linking through route search state.

## Goals

- Define the minimal behavior retained for the `/agents` route family.
- Preserve shareable selected-agent route state while broader `/agents`
  behavior is re-specified later.

## Non-Goals

- Defining fleet listing, empty states, or session posture in detail.
- Defining new-run authoring or run-control behavior.

## Route Contract

- The `/agents` route family MUST inherit the application-shell layout contract
  defined in `PRD-0150`.
- The `/agents` route family MUST support selected-agent deep-linking through
  the `agent` search parameter.
- Opening `/agents?agent=<agent-id>` MUST preserve the selected-agent intent in
  a refresh-stable and shareable route shape.

## Acceptance Scenarios

### Scenario SCN-0000: Deep-links the selected agent in the agents route

Given the application can resolve a valid agent identity  
When the operator opens `/agents?agent=<agent-id>`  
Then the application preserves that selected-agent intent in the `/agents`
route state.
