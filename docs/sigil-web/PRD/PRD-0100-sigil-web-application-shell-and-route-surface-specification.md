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
- Define the current standalone run-detail placeholder surface without
  inventing future behavior.

## Non-Goals

- Defining root workspace selection behavior.
- Defining live app-server session behavior.
- Defining future standalone run-detail functionality beyond the placeholder.

## Shared Shell Contract

- The document shell MUST wrap route content in the shared `sigil-web`
  application shell.
- The shared shell MUST expose `data-testid="app-shell"` and
  `data-testid="app-shell-main"` as stable selectors.
- The shared shell MUST remain visible on both `/` and `/runs/$runId`.

## Route Surface Contract

- `/` MUST render the root agent workspace within the shared shell.
- `/runs/$runId` MUST render a standalone workspace container within the shared
  shell.
- The standalone `/runs/$runId` route remains a placeholder until a future PRD
  replaces that contract.

## Deferred Contracts

- The standalone `/runs/$runId` route does not yet define tab content,
  projection loading, or interactive run controls.

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

### Scenario SCN-0002: Exposes the standalone run-detail route as a placeholder workspace frame

Given a user opens `/runs/$runId` in `sigil-web`  
When the standalone route renders  
Then the route exposes the current placeholder workspace frame and scroll
region without additional run-detail content.
