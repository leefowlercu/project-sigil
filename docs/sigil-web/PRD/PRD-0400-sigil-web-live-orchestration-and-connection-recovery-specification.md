# PRD-0400: Sigil-Web Live Orchestration and Connection Recovery Specification

## Status

Draft

## Context

Beyond static inspection, `sigil-web` needs to observe active runs in-place and
resume that observation cleanly when the session connection goes stale.

## Goals

- Define live run subscription behavior for the web workspace.
- Define resume behavior after reconnect.
- Define terminal-state transitions for live workspaces.

## Non-Goals

- Defining the initial session handshake contract in full.
- Defining inline YAML authoring rules in detail.

## Live Subscription Contract

- Live run detail MUST attach through `run/subscribe`.
- Canonical `run/eventAppended` notifications remain the source-of-truth live
  feed for incremental updates.
- Higher-level notifications MAY enrich the UI but MUST NOT replace canonical
  event application.
- The live workspace MUST surface operator-visible evidence that the view is
  actively receiving updates.

## Resume Contract

- Reconnect recovery MUST resume live subscriptions with `afterSeq` derived
  from the highest applied canonical event.
- Resume MUST NOT duplicate already-applied canonical events in the operator
  timeline.
- Reconnect recovery MUST preserve the current run workspace context.

## Terminal Transition Contract

- A live workspace MUST transition cleanly when the subscribed run reaches
  completed or interrupted terminal state.
- Terminal-state UI MUST expose the resulting status and terminal summary
  without requiring the operator to open a second page.

## Acceptance Scenarios

### Scenario SCN-0000: Attaches a live run workspace to canonical subscription updates

Given the session is ready and one selected run is still active  
When the operator attaches the run detail workspace to live updates  
Then the application applies canonical subscription updates within that
workspace as the run progresses.

### Scenario SCN-0001: Resumes a live run workspace after reconnect without duplicating applied events

Given the operator is viewing a live run workspace that has already applied
canonical events  
When the connection degrades, reconnects, and resumes the subscription  
Then the application resumes from the highest applied sequence without
duplicating already-applied events.

### Scenario SCN-0002: Transitions a subscribed run workspace into terminal completed or interrupted state

Given the operator is viewing a live run workspace for an active run  
When the subscribed run reaches completed or interrupted terminal state  
Then the application transitions the workspace into the matching terminal state
and preserves terminal context for review.
