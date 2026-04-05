# PRD-0100: Sigil-Web Application Shell and Route Surface Specification

## Status

Accepted

## Context

`sigil-web` currently exposes a shared application shell around two routes:

- `/`: the root agent workspace
- `/runs/$runId`: a standalone run-detail placeholder

The bootstrap suite needs a durable shell and route-surface contract before
behavior can be expanded safely.

## Goals

- Define the shared application shell contract across the current route surface.
- Define the standalone run-detail route surface contract including live agent
  session support via query parameter.

## Non-Goals

- Defining root workspace selection behavior.
- Defining live app-server session behavior.
- Defining run-detail step inspection content (see PRD-0240).

## Shared Shell Contract

- The document shell MUST wrap route content in the shared `sigil-web`
  application shell.
- The shared shell MUST expose `data-testid="app-shell"` and
  `data-testid="app-shell-main"` as stable selectors.
- The shared shell MUST remain visible on both `/` and `/runs/$runId`.

## Route Surface Contract

- `/` MUST render the root agent workspace within the shared shell.
- `/runs/$runId` MUST render the standalone run-detail workspace within the
  shared shell.
- The standalone `/runs/$runId` route MUST accept an optional `agent` query
  parameter to identify the live agent session for run data.
- The `Open Detail` affordance on the root workspace MUST navigate to
  `/runs/$runId?agent=$agentId` for the selected run.

## Acceptance Scenarios

### Scenario SCN-0000: Wraps the root agent workspace route in the shared sigil-web application shell

Given a user opens the root `sigil-web` route  
When the root agent workspace renders  
Then the page exposes `app-shell` and `app-shell-main` around the route
content.

### Scenario SCN-0001: Preserves the shared sigil-web application shell on the standalone run-detail route

Given a user opens `/runs/$runId` in `sigil-web`  
When the standalone route renders  
Then the page still exposes `app-shell` and `app-shell-main`.

### Scenario SCN-0002: Renders the standalone run-detail workspace with the agent query parameter

Given the root workspace shows embedded run detail for a selected run  
When a user activates `Open Detail`  
Then the browser navigates to `/runs/$runId` with the agent query parameter
identifying the live agent session.
