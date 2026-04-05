# PRD-0240: Sigil-Web Run Detail Step Inspection Specification

## Status

Accepted

## Context

The standalone `/runs/$runId` route renders a Steps Pane (PRD-0230) in the left
sidebar. The right content area is currently empty. Operators need to select an
individual step and inspect its metadata, accounting, executed code, and action
output within the run-detail page.

## Goals

- Define the step selection contract between the Steps Pane and the right
  content area.
- Define the Step Context Pane header contract for step metadata, accounting,
  and action pagination.
- Define the Code Pane contract for syntax-highlighted action source display.
- Define the Action Output Pane contract for action execution results.
- Define the empty state when the selected step has no actions.

## Non-Goals

- Defining step-level editing or re-execution controls.
- Defining model turn or user turn content display.

## Step Selection Contract

- Selecting a step card in the Steps Pane MUST populate the right content area
  with that step's detail.
- When steps are available and no step has been manually selected, the first
  step MUST be auto-selected on page load.
- The right content area MUST show an empty prompt when no step is selected and
  no steps are available.

## Step Context Pane Contract

- The Step Context Pane MUST display the step number, node ID, status, and
  duration.
- The Step Context Pane MUST display step-level accounting when available,
  including token counts and cost.
- When the selected step has more than one action, the Step Context Pane MUST
  display an action pagination control to navigate between actions.
- When the selected step has zero or one action, no pagination control is
  displayed.

## Code Pane Contract

- The Code Pane MUST display the executed source code from the selected action
  artifact with syntax highlighting appropriate to the action language.
- When the selected step has no actions, the Code Pane MUST display an empty
  state.

## Action Output Pane Contract

- The Action Output Pane MUST always display stdout and stderr sections from the
  selected action artifact, showing a placeholder when either is empty.
- When the action has error detail, the Action Output Pane MUST display the
  error code, message, and structured error detail.
- When the action has subcall traces, the Action Output Pane MUST display the
  subcall list with index, type, status, duration, provider, and model.
- When the selected step has no actions, the Action Output Pane MUST display an
  empty state.

## Live Update Contract

- When viewing an in-progress step, the Step Context Pane MUST dynamically
  update the step status and duration as the step transitions from running to
  completed.
- When an action completes during a running step, the Code Pane and Action
  Output Pane MUST update to display the action content.

## Acceptance Scenarios

### Scenario SCN-0000: Displays step detail in the right content area when a step is selected in the Steps Pane

Given the standalone run-detail page is open for a completed run  
When the operator selects a step in the Steps Pane  
Then the right content area displays the selected step's context pane, code
pane, and action output pane.

### Scenario SCN-0001: Shows an empty prompt in the right content area when no step is selected

Given the standalone run-detail page is open for a completed run  
When no step is selected in the Steps Pane  
Then the right content area displays an empty step-selection prompt.

### Scenario SCN-0002: Displays step metadata and accounting in the Step Context Pane

Given a step is selected in the standalone run-detail page  
When the Step Context Pane renders  
Then the Step Context Pane displays the step number, node ID, status, duration,
and available accounting information.

### Scenario SCN-0003: Displays action pagination when the selected step has multiple actions

Given a step with multiple actions is selected in the standalone run-detail page  
When the Step Context Pane renders  
Then the Step Context Pane displays a pagination control showing the current
action index and total action count.

### Scenario SCN-0004: Displays syntax-highlighted source code in the Code Pane

Given a step with at least one action is selected in the standalone run-detail
page  
When the Code Pane renders  
Then the Code Pane displays the action source code with syntax highlighting
appropriate to the action language.

### Scenario SCN-0005: Displays stdout and stderr in the Action Output Pane

Given a step with at least one action is selected in the standalone run-detail
page  
When the Action Output Pane renders  
Then the Action Output Pane displays the action stdout and stderr.

### Scenario SCN-0006: Shows an empty state in the code and output panes when the selected step has no actions

Given a step with no actions is selected in the standalone run-detail page  
When the code and output panes render  
Then both panes display an empty state indicating no action was executed.

### Scenario SCN-0007: Auto-selects the first step when the run-detail page loads with steps available

Given the standalone run-detail page is open for a completed run with steps  
When the page finishes loading  
Then the first step is automatically selected and its detail is displayed.

### Scenario SCN-0008: Displays subcall traces in the Action Output Pane when the action has subcalls

Given a step with an action containing subcalls is selected in the standalone
run-detail page  
When the Action Output Pane renders  
Then the Action Output Pane displays the subcall list with index, type, status,
and duration.

### Scenario SCN-0009: Dynamically updates step status in the Step Context Pane when the selected step completes

Given the standalone run-detail page is open for an in-progress run  
When the selected step transitions from running to completed  
Then the Step Context Pane updates to reflect the completed status and final
duration.
