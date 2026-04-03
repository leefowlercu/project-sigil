# PRD-0230: Sigil-Web Run Detail Page Steps Pane Specification

## Status

Accepted

## Context

The standalone `/runs/$runId` route currently renders a placeholder workspace
frame (PRD-0100 SCN-0002). The run-detail page needs a Steps Pane in the left
sidebar that presents all steps from a run's node tree with depth-based
indentation, supports live updates for in-progress runs, and displays a header
with the current step count.

## Goals

- Define the Steps Pane step-rendering contract for the standalone run-detail
  page.
- Define depth-based indentation behavior for steps across the run's node tree.
- Define the Steps Pane header counter contract.
- Define live update behavior for in-progress runs.

## Non-Goals

- Defining the right-side content area of the standalone run-detail page.
- Defining step selection or interaction behavior.
- Defining the embedded run-detail pane in the root workspace.

## Steps Pane Rendering Contract

- The standalone run-detail page MUST render a Steps Pane in the left sidebar
  container.
- The Steps Pane MUST display all steps from the run, including steps from the
  root node and all descendant nodes.
- Steps MUST be ordered by their position in the global step timeline.

## Depth Indentation Contract

- Steps on the root node MUST render at indentation level 0.
- Steps on child nodes of the root node MUST render at indentation level 1.
- Steps on deeper descendant nodes MUST render at indentation levels
  commensurate with their node depth in the run tree.

## Steps Pane Header Contract

- The Steps Pane MUST display a header with an icon, label, and step count
  badge.
- The step count badge MUST reflect the number of steps currently displayed.

## Live Update Contract

- When viewing an in-progress run, the Steps Pane MUST dynamically append new
  steps as they occur.
- The step count badge MUST update to reflect newly appended steps.

## Acceptance Scenarios

### Scenario SCN-0000: Shows all run steps from the root node and all descendant nodes in the run-detail Steps Pane

Given the standalone run-detail page is open for a completed run  
When the Steps Pane renders  
Then the Steps Pane displays steps from the root node and all descendant
nodes.

### Scenario SCN-0001: Indents run-detail steps based on their node depth in the run tree

Given the standalone run-detail page is open for a completed run with nested
nodes  
When the Steps Pane renders  
Then root-node steps render at indentation level 0 and descendant-node steps
render at indentation levels matching their node depth.

### Scenario SCN-0002: Displays the current step count in the run-detail Steps Pane header badge

Given the standalone run-detail page is open for a completed run  
When the Steps Pane renders  
Then the Steps Pane header badge shows the total number of steps displayed.

### Scenario SCN-0003: Dynamically appends new steps and updates the header count for an in-progress run in the run-detail Steps Pane

Given the standalone run-detail page is open for an in-progress run  
When new steps are appended during execution  
Then the Steps Pane appends the new steps and the header badge count updates
to reflect the current total.
