# PRD-0300: Sigil-Web Run Detail Route Specification

## Status

Draft

## Context

The `/runs/$runId` route family remains the owner of the run-detail interaction
contract in `sigil-web`. The first shipped run-detail surface is currently the
shared run-detail pane embedded in `/agents`, but that embedding does not
change the owning route family or the operator-visible behavior that future
dedicated run-detail routes must preserve.

## Goals

- Define the live timeline interaction contract for run-detail surfaces.
- Preserve a single owning PRD for run-detail behavior while the dedicated
  `/runs/$runId` route family is still being populated.
- Keep embedded run-detail surfaces behaviorally aligned with the future route
  family owner.

## Non-Goals

- Defining backend pagination or partial-history event loading.
- Defining run-start, run-stop, or other orchestration controls.
- Defining the full visual layout of every run-detail tab beyond the live
  timeline interaction contract.

## Route Contract

- The `/runs/$runId` route family MUST inherit the application-shell layout
  contract defined in `PRD-0150`.
- The run-detail timeline MUST keep the latest event visible while a run is
  live and the operator has not scrolled away from the bottom of the timeline.
- When the operator scrolls above the latest event in an overflowing timeline,
  the run-detail timeline MUST reveal a centered floating `Scroll to bottom`
  control near the bottom edge of the pane.
- Activating the `Scroll to bottom` control in a live run MUST return the
  timeline to the latest event and re-arm automatic live following.
- Embedded run-detail surfaces, including `/agents` renderings of the shared
  run-detail pane, MUST preserve this same interaction contract until the
  dedicated `/runs/$runId` route family is populated.

## Acceptance Scenarios

### Scenario SCN-0000: Auto-follows the latest event while a live run detail timeline is pinned to the bottom

Given the operator is viewing a live run detail timeline at its latest event  
When new live run events append to the timeline  
Then the timeline keeps the latest event visible without additional operator
input.

### Scenario SCN-0001: Shows a centered scroll-to-bottom control and resumes follow when the operator has scrolled away from the latest live event

Given the operator is viewing an overflowing live run detail timeline above its
latest event  
When the operator has scrolled away from the bottom of the timeline  
Then the timeline shows a centered floating `Scroll to bottom` control near the
bottom of the pane  
And activating that control returns the timeline to the latest event  
And the timeline resumes automatic live following for subsequent appended
events.
