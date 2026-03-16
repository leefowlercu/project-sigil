# PRD-0300: Sigil-Web Run Detail Workspace Specification

## Status

Draft

## Context

The core operator workflow in `sigil-web` is inspecting a run in one workspace
that combines run summary, topology, time-ordered progress, and typed artifact
detail.

## Goals

- Define the baseline run detail workspace layout and behavior.
- Define how read-plane data populates summary, tree, timeline, step, and
  artifact panes.
- Preserve current drill-in context during refreshes.

## Non-Goals

- Defining live subscription recovery in detail.
- Defining run start or stop control UX in detail.

## Workspace Contract

- The run detail route MUST load its primary summary from the app-server read
  surface.
- The workspace MUST expose:
  - run summary
  - node tree
  - run-global timeline or step progression
  - step detail
  - typed artifact detail
- Drill-in interactions SHOULD keep the operator inside the current run
  workspace rather than navigating to detached detail routes.

## Refresh Contract

- Refreshing run detail data MUST keep the active run selection stable.
- Refreshing run detail data MUST preserve selected node, step, and artifact
  context when the underlying entities still exist.
- Read refreshes MUST not require the operator to rebuild the entire workspace
  mentally after each action.

## Acceptance Scenarios

### Scenario SCN-0000: Opens a run detail workspace with summary tree and timeline panes

Given the session is ready and one persisted run is selected  
When the operator opens that run's detail workspace  
Then the application shows summary, tree, and timeline panes for that run in a
single workspace.

### Scenario SCN-0001: Selects a node step and typed artifact without leaving the current run workspace

Given the operator is viewing a run detail workspace with available node and
step data  
When the operator selects a node, then a step, then a typed artifact  
Then the application shows the selected detail without leaving the current run
workspace.

### Scenario SCN-0002: Refreshes run detail data without losing the current selection context

Given the operator is viewing a run detail workspace with an active node step
or artifact selection  
When the operator refreshes the run detail data  
Then the application preserves the current selection context when that data is
still valid.
