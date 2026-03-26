# PRD-0210: Sigil-Web Run Detail Workspace Specification

## Status

Accepted

## Context

The root `sigil-web` workspace embeds run-detail inspection beside the run list,
and the current UI also exposes an `Open Detail` transition to a standalone
placeholder route.

## Goals

- Define the current root embedded run-detail contract.
- Define the current empty state when no active run is selected.
- Define the navigation from the root workspace to the standalone placeholder
  route.

## Non-Goals

- Defining a future standalone run-detail implementation beyond the placeholder.
- Defining run execution or stop behavior.

## Embedded Run Detail Contract

- When the selected agent has an active run selection, the root workspace MUST
  render embedded run detail for that run.
- Embedded run detail MUST expose the current tab surface for timeline, nodes,
  steps, and meta information.
- The root workspace MUST show the current empty prompt when there is no active
  run to inspect.

## Navigation Contract

- The embedded `Open Detail` affordance MUST navigate to the standalone
  `/runs/$runId` route for the selected run.

## Acceptance Scenarios

### Scenario SCN-0000: Loads the selected run detail in the root workspace when the selected agent has runs

Given the selected root workspace agent has one or more runs  
When the workspace renders  
Then the embedded run detail shows the current run with the timeline, nodes,
steps, and meta tabs.

### Scenario SCN-0001: Shows an empty run-detail prompt when the selected agent has no runs

Given the selected root workspace agent has no runs  
When the workspace renders  
Then the root workspace shows the current empty run-detail prompt instead of an
embedded run detail view.

### Scenario SCN-0002: Opens the selected run in the standalone run-detail route from the root workspace

Given the root workspace shows embedded run detail for a selected run  
When a user activates `Open Detail`  
Then the browser navigates to `/runs/$runId` for that run.
