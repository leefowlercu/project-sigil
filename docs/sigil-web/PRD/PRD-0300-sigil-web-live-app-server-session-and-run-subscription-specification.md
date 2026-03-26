# PRD-0300: Sigil-Web Live App-Server Session and Run Subscription Specification

## Status

Accepted

## Context

In live mode, `sigil-web` connects to `sigil` app-server endpoints over
WebSocket, negotiates a supported protocol version, tracks connection state from
heartbeat activity, and updates the root workspace from live run notifications.

## Goals

- Define the live agent connection lifecycle visible through the root workspace.
- Define the live run snapshot and status-notification behavior visible through
  the root workspace.

## Non-Goals

- Defining direct browserless protocol acceptance.
- Defining visual review workflows or manifest layers.

## Live Session Contract

- Connecting a live endpoint MUST create a visible fleet record in the root
  workspace.
- A successful initialize handshake MUST transition the live agent session to
  the ready state.
- Missed heartbeats MUST transition the visible live session to degraded until
  the session reconnects or disconnects.
- Reconnect from the root workspace MUST attempt to restore the live session.
- Remove from the root workspace MUST delete the live session from the visible
  fleet.

## Live Runs Contract

- When a live session is active, the root workspace MUST load the current runs
  snapshot for the selected agent.
- `runs/changed`, `run/statusChanged`, and `run/completed` notifications MUST
  update the visible run list and embedded detail state for the selected agent.

## Acceptance Scenarios

### Scenario SCN-0000: Connects a live agent endpoint and exposes a ready agent session in the fleet

Given `sigil-web` is running in live mode  
When a user connects a reachable app-server WebSocket endpoint  
Then the endpoint appears in the fleet and the session becomes ready after
initialize.

### Scenario SCN-0001: Marks a live agent session degraded after missed heartbeats and recovers on reconnect

Given a live agent session has become ready in the fleet  
When the app-server stops sending heartbeats long enough to miss the heartbeat
window  
Then the session becomes degraded  
When a user requests reconnect after the server resumes heartbeats  
Then the session returns to ready.

### Scenario SCN-0002: Applies live runs snapshot and run status notifications to the selected agent workspace

Given a ready live agent session is selected in the root workspace  
When the app-server serves an initial runs snapshot and later emits run status
changes  
Then the selected agent workspace updates the visible run list and embedded run
detail state.

### Scenario SCN-0003: Removes a live agent session from the fleet when the operator requests removal

Given a live agent session is visible in the fleet  
When a user removes that agent session  
Then the fleet no longer shows the removed session.
