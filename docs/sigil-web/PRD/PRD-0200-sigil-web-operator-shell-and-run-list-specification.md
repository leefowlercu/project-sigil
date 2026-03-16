# PRD-0200: Sigil-Web Operator Shell and Run List Specification

## Status

Draft

## Context

Once the session is connected, operators need one predictable shell for
navigation and one readable run index for discovering work to inspect or
control.

## Goals

- Define the primary operator shell for the web UI.
- Define run list loading, pagination, and empty-state behavior.
- Keep connection context visible while browsing the run corpus.

## Non-Goals

- Defining live run subscription semantics.
- Defining inline YAML authoring or stop control.

## Operator Shell Contract

- The operator shell MUST expose global session health and primary navigation.
- The shell MUST preserve connection context while operators move between run
  list, run detail, and run authoring workflows.
- The run list workflow MUST use the shared app-server read surface rather than
  a second cached source of truth.

## Run List Contract

- The run list view MUST show persisted runs after session initialization.
- Pagination MUST be explicit and deterministic when additional runs exist.
- Pagination actions MUST NOT discard current session context or downgrade
  session health indicators.
- Empty-state UI MUST provide a clear next action for operators when the run
  corpus is empty.

## Acceptance Scenarios

### Scenario SCN-0000: Lists persisted runs after session initialization

Given the session is ready and the app-server run corpus contains persisted
runs  
When the operator opens the run list view  
Then the application shows the persisted runs together with current connection
context.

### Scenario SCN-0001: Paginates the run list without losing current connection context

Given the session is ready and the run corpus spans multiple pages  
When the operator requests the next page of runs  
Then the application loads the next page while preserving global session
context and navigation state.

### Scenario SCN-0002: Shows an empty-state call to action when the run corpus has no runs

Given the session is ready and the app-server run corpus is empty  
When the operator opens the run list view  
Then the application shows an empty-state call to action that can lead the
operator toward creating or starting a run.
