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

- Connecting a live endpoint MUST keep the pending session hidden from the
  visible fleet until `initialize` succeeds and returns the real Agent Instance
  ID.
- A successful initialize handshake MUST transition the live agent session to
  the ready state and surface a visible fleet record keyed by the exact Agent
  Instance ID.
- Missed heartbeats MUST transition the visible live session to degraded until
  the session reconnects or disconnects.
- Reconnect from the root workspace MUST attempt to restore the live session.
- Remove from the root workspace MUST delete the live session from the visible
  fleet.
- When a newly initialized live session resolves to an Agent Instance ID that
  is already visible in the fleet, `sigil-web` MUST reject the duplicate
  connection, keep the current workspace selection unchanged, and surface a
  warning toast to the operator.

## Live Runs Contract

- When a live session is active, the root workspace MUST load the current runs
  snapshot for the selected agent.
- `runs/changed`, `run/statusChanged`, and `run/completed` notifications MUST
  update the visible run list and embedded detail state for the selected agent.

## Acceptance Scenarios

### Scenario SCN-0000: Connects a live agent endpoint only after initialize resolves a canonical Agent Instance ID

Given `sigil-web` is running in live mode  
When a user connects a reachable app-server WebSocket endpoint  
Then the endpoint stays hidden from the visible fleet until initialize
completes  
And the canonical live agent appears in the fleet and becomes ready after
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

### Scenario SCN-0004: Rejects a duplicate live agent connection that resolves to an already-visible Agent Instance ID

Given a live agent session is visible in the fleet  
When a user connects another reachable app-server WebSocket endpoint whose
initialize response returns the same Agent Instance ID  
Then the fleet still shows only one visible agent with that Agent Instance ID  
And the current workspace selection remains unchanged  
And the operator sees a warning toast that the duplicate agent cannot be added.
