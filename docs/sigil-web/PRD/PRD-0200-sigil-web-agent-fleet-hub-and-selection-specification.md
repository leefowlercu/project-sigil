# PRD-0200: Sigil-Web Agent Fleet Hub and Selection Specification

## Status

Draft

## Context

Once the session is connected, operators need one predictable command hub for
fleet visibility, agent selection, and agent-scoped run discovery.

## Goals

- Define the `/agents` command hub for the web UI.
- Define fleet listing, selected-agent detail, and selected-agent run list
  behavior.
- Keep selected-agent state deep-linkable without losing connection context.

## Non-Goals

- Defining live run subscription semantics.
- Defining inline YAML dialog behavior or stop control in detail.

## Agent Hub Contract

- The root path `/` MUST redirect to `/agents`.
- The `/agents` hub MUST expose global session health, fleet status, selected
  agent detail, and selected-agent run list in one workspace.
- The hub MUST preserve connection context while operators move between agent
  selection, run detail, and run authoring workflows.
- The selected agent MUST be deep-linkable through the route search parameter
  `agent` at `/agents?agent=<agent-id>`.
- Selecting a different agent through the UI MUST update the route search
  parameter so the current hub state remains shareable and refresh-stable.

## Fleet and Agent Detail Contract

- The hub MUST show connected agents after session initialization.
- When an agent is selected, the hub MUST show operator-visible agent details
  and that agent's runs without requiring a second list page.
- Agent-scoped run lists MUST use the shared app-server read surface rather
  than a second cached source of truth.
- Empty-state UI MUST provide a clear next action when no agents are connected
  to the app-server.

## Acceptance Scenarios

### Scenario SCN-0000: Lists connected agents in the command hub after session initialization

Given the session is ready and the app-server exposes connected agents  
When the operator opens `/agents`  
Then the application shows the connected fleet together with current connection
context.

### Scenario SCN-0001: Deep-links the selected agent in the agents hub and shows its details plus runs

Given the session is ready and multiple connected agents are available  
When the operator opens `/agents?agent=<agent-id>`  
Then the application selects that agent in the hub and shows its details plus
runs.

### Scenario SCN-0002: Shows a fleet empty state when no agents are connected to the app-server

Given the session is ready and no agents are connected to the app-server  
When the operator opens `/agents`  
Then the application shows a fleet empty state with a clear next action for
restoring or awaiting agent connectivity.
