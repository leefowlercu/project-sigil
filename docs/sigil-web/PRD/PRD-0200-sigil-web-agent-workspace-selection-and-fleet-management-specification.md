# PRD-0200: Sigil-Web Agent Workspace Selection and Fleet Management Specification

## Status

Accepted

## Context

The root `sigil-web` workspace presents the operator's agent fleet, keeps the
selected agent deep-linkable with `?agent=`, and allows the fleet list to be
filtered before an operator drills into runs.

## Goals

- Define how root workspace agent selection is resolved.
- Define the current filtering behavior for fleet cards.

## Non-Goals

- Defining live WebSocket session lifecycle details.
- Defining new-run execution behavior.

## Selection Contract

- The root workspace MUST default to the first available agent when no valid
  `?agent=` search value is present.
- The selected agent MUST be reflected through the `agent` search parameter
  using the exact canonical Agent Instance ID.
- Canonical selection resolution MUST use the exact Agent Instance ID for a
  visible agent.
- Invalid or legacy `?agent=` values, including historical `agent_...` forms,
  MUST fall back to the first available agent instead of resolving to a
  selected agent.
- Choosing a different agent card MUST update the selected workspace agent.

## Fleet Filter Contract

- The fleet filter MUST match agent cards by instance name or endpoint.
- Filtering MUST only affect the visible fleet list; it MUST NOT mutate the
  underlying fleet state.

## Related Contracts

- New-run authoring behavior for the selected root workspace agent is defined
  by `PRD-0220`.

## Acceptance Scenarios

### Scenario SCN-0000: Defaults the root workspace selection to the first available agent and reflects its canonical Agent Instance ID in the agent search param

Given the root workspace has one or more visible agent cards  
When a user opens `/` without an `agent` search parameter  
Then the first available agent becomes selected and the hidden agent search
value reflects that selection.

### Scenario SCN-0001: Resolves exact canonical Agent Instance IDs and falls back when the agent search value is invalid or legacy

Given the root workspace exposes one or more visible agent cards  
When a user opens `/` with an `agent` search value that matches a visible
agent's exact Agent Instance ID  
Then that same agent card is selected  
When a user opens `/` with an invalid or legacy `agent` search value for a
different agent  
Then the first available agent card becomes selected.

### Scenario SCN-0002: Updates root workspace selection when a different agent card is chosen

Given the root workspace shows more than one agent card  
When a user selects a different agent card  
Then the selected agent panel and hidden agent search value update to that
agent.

### Scenario SCN-0003: Filters agent cards by instance name or endpoint without mutating the underlying fleet

Given the root workspace shows one or more agent cards  
When a user applies a fleet filter by instance name or endpoint  
Then only matching agent cards remain visible while the underlying fleet count
remains unchanged.
