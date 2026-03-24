# PRD-0190: Sigil App-Server Execution Subscriptions and Control Specification

## Status

Accepted

## Context

After the transport core and shared read plane land, the app-server needs a
run-execution contract that can start runs from inline YAML, attach to active
runs, resume via canonical event sequence, and preserve current graceful-stop
semantics for remote clients.

The implemented app-server execution and control plane now exposes:

- `runs/list`
- `run/start`
- `runs/subscribe`
- `runs/unsubscribe`
- `run/subscribe`
- `run/unsubscribe`
- `run/stop`
- `runs/changed`
- canonical event replay and resume via `afterSeq`
- app-level live notifications layered on canonical event delivery

## Goals

- Define the v1 JSON-RPC execution, subscription, and stop-control surface.
- Preserve current runtime authority and stop semantics across remote control.
- Keep canonical per-run event `seq` as the resume and de-duplication key.
- Define one instance-level live run-summary subscription surface for rich
  clients that need the current state of the configured app-server run corpus.
- Keep app-server control handlers thin over shared runtime and control
  services instead of inventing a second run-control stack.

## Non-Goals

- Defining auth, approvals, or non-localhost deployment policy.
- Defining post-v1 hardening and ergonomics follow-on work.
- Defining a second interruption model separate from `run/stop`.

## Run Start Contract

- `run/start` MUST accept inline `runConfigYaml` and MAY accept
  `templateVars`.
- `run/start` MUST NOT require server-local config file paths.
- `run/start` success MUST be tied to durable `run.queued` persistence rather
  than a later `run.running` transition.
- `run/start` MUST return:
  - `runId`
  - `asOfSeq`
  - `payload.run`
- The initial `payload.run.state` MUST be `queued`.
- App-server-started runs MUST persist `run.queued.source=app_server.run.start`.
- App-server-started runs without a real config path MUST omit
  `runConfigPath`.
- The submitted inline YAML MUST be persisted as a run-local provenance
  artifact.
- Runs started through the app-server MUST continue independently of the client
  connection lifetime.

## Subscription Contract

- `runs/subscribe` MUST return one fresh authoritative snapshot in
  `payload.items` plus the collection `payload.revision`.
- `runs/subscribe` fresh snapshot ordering MUST match `runs/list`
  newest-first semantics.
- `runs/subscribe` MUST return the full configured app-server run corpus rather
  than a paged or filtered subset.
- `runs/changed` MUST carry one monotonic `revision` plus one typed collection
  delta in `payload.kind`.
- `runs/changed` `payload.kind=upsert` MUST carry one full `RunSummaryView` for
  each new or changed run summary detected after subscription starts.
- `runs/changed` `payload.kind=remove` MUST carry the removed `runId` when one
  previously indexed run disappears from the configured corpus.
- `runs/changed` `payload.kind=reset` MUST instruct clients to fetch one fresh
  authoritative snapshot before applying later deltas.
- `runs/changed` MUST be derived from the shared run-summary read path rather
  than from ad hoc mutation of notification payload state.
- `runs/changed` MUST NOT require a concurrent `run/subscribe` for the same
  run.
- Duplicate `runs/subscribe` calls on one connection MUST replace the earlier
  instance-level subscription instead of creating duplicate live delivery.
- `runs/unsubscribe` MUST be idempotent and connection-local.
- Clients that detect a stale idle connection through a missed-heartbeat window
  MUST be able to reconnect, reinitialize, and issue a fresh `runs/subscribe`
  request to receive a new authoritative snapshot before subsequent
  `runs/changed` delivery.
- `runs/subscribe` and `runs/changed` MUST NOT define resume cursors,
  canonical-event replay, or missed-update replay in v1.
- `run/subscribe` MUST support:
  - fresh attach without `afterSeq`
  - resume attach with `afterSeq`
- Fresh attach MUST return:
  - `snapshotAsOfSeq`
  - `payload.snapshot`
  - `payload.terminal`
- Resume attach MUST return:
  - `snapshotAsOfSeq`
  - `payload.replayEvents`
  - `payload.terminal`
- Resume attach MUST replay only canonical events with `seq > afterSeq`.
- `snapshotAsOfSeq` MUST come from one authoritative event scan used to derive
  the fresh snapshot.
- Live delivery MUST keep canonical `run/eventAppended` notifications ordered
  by increasing `seq` for each subscribed run.
- Live delivery for app-server-owned active runs MUST prefer the runner's
  in-process event observer path when available.
- Live delivery for runs started outside the current app-server process MUST
  continue to work through persisted-event replay and polling within the
  configured app-server run corpus.
- Idle WebSocket connections MUST emit app-level `server/heartbeat`
  notifications.
- Clients that detect a stale idle connection through a missed-heartbeat window
  MUST be able to reconnect, reinitialize, and resume `run/subscribe` with
  `afterSeq` without gaps for canonical events with `seq > afterSeq`.
- Higher-level notifications such as `run/started`, `run/statusChanged`, and
  `run/completed` MAY be emitted, but canonical `run/eventAppended` remains the
  source-of-truth live update.
- Subscribing to an already-terminal run MUST return a terminal snapshot or
  replay result instead of an error.
- Duplicate `run/subscribe` calls for the same run on one connection MUST
  replace the earlier subscription instead of creating duplicate live delivery.
- `run/unsubscribe` MUST be idempotent and connection-local.

## Stop Contract

- `run/stop` MUST target only runs within the configured app-server run corpus.
- `run/stop` MUST reuse the existing `sigil run stop` semantics over:
  - canonical `events.jsonl` validation
  - terminal no-op behavior
  - `process.json` validation
  - `stop-request.json` persistence
  - `run.interrupted` terminalization semantics
- Terminal runs MUST return success with `payload.stopRequested=false`.
- Non-terminal runs MUST persist `stop-request.json` with:
  - `requested_by=app_server.run.stop`
  - `signal=SIGTERM`
- In-process app-server-owned runs MAY use an internal cancellation fast path,
  but they MUST still preserve the same canonical runtime outcome and persisted
  stop-request metadata.
- Successful `run/stop` responses MUST return:
  - `runId`
  - `payload.stopRequested`
  - `payload.state`
  - `payload.eventsPath`
- User-request interruption caused by app-server stop MUST persist
  `run.interrupted.reason=user_request`.
- User-request interruption caused by app-server stop MUST persist
  `run.interrupted.interrupted_by=app_server.run.stop`.

## Error Contract

- Invalid inline YAML for `run/start` MUST return stable domain code
  `invalid_run_config_yaml`.
- Invalid resume inputs for `run/subscribe` MUST return stable domain code
  `invalid_resume_cursor`.
- Missing run targets for `run/subscribe` or `run/stop` MUST return stable
  domain code `run_not_found`.
- Missing non-terminal process metadata for `run/stop` MUST return stable
  domain code `process_metadata_unavailable`.
- Stale process metadata for `run/stop` MUST return stable domain code
  `stale_process_metadata`.

## Acceptance Scenarios

### Scenario SCN-0000: Starts a queued app-server run from inline YAML after initialize handshake on stdio

Given a client completes `initialize` and `initialized` on the stdio transport  
When the client requests `run/start` with inline `runConfigYaml`  
Then the server returns a queued-state response with `runId`, `asOfSeq`, and
`payload.run.source=app_server.run.start`.

### Scenario SCN-0001: Serves fresh attach snapshot for run subscribe after initialize handshake on stdio

Given one persisted run exists in the configured app-server run directory  
When a client completes `initialize` and `initialized` on the stdio transport  
And the client requests `run/subscribe` without `afterSeq`  
Then the server returns `snapshotAsOfSeq`, `payload.snapshot`, and terminal
fresh-attach state for that run.

### Scenario SCN-0002: Replays canonical events for run subscribe resume attach after initialize handshake on stdio

Given one persisted run exists in the configured app-server run directory  
When a client completes `initialize` and `initialized` on the stdio transport  
And the client requests `run/subscribe` with `afterSeq`  
Then the server replays only canonical events with `seq > afterSeq`.

### Scenario SCN-0003: Unsubscribes idempotently after initialize handshake on stdio

Given one persisted run exists in the configured app-server run directory  
When a client completes `initialize` and `initialized` on the stdio transport  
And the client requests `run/unsubscribe` twice for the same run  
Then the first response reports `unsubscribed=true`  
And the second response reports `unsubscribed=false`.

### Scenario SCN-0004: Returns terminal no-op run stop results after initialize handshake on stdio

Given one completed run exists in the configured app-server run directory  
When a client completes `initialize` and `initialized` on the stdio transport  
And the client requests `run/stop` for that run  
Then the server returns success with `payload.stopRequested=false` and
`payload.state=completed`.

### Scenario SCN-0005: Delivers live canonical notifications for subscribed runs after initialize handshake on stdio

Given one non-terminal run exists in the configured app-server run directory  
When a client completes `initialize` and `initialized` on the stdio transport  
And the client subscribes to that run  
Then the client receives canonical live notifications as the run progresses.

### Scenario SCN-0006: Replaces duplicate run subscribe requests without duplicate live delivery after initialize handshake on stdio

Given one non-terminal run exists in the configured app-server run directory  
When a client completes `initialize` and `initialized` on the stdio transport  
And the client subscribes twice to that run on the same connection  
Then the server replaces the earlier subscription  
And the client still receives one canonical `run/eventAppended` stream without
duplicate `seq` delivery.

### Scenario SCN-0007: Stops an actively running local CLI run after initialize handshake on stdio

Given a local CLI run is actively executing  
When a client completes `initialize` and `initialized` on the stdio transport  
And the client requests `run/stop` for that run  
Then the server returns success with `payload.stopRequested=true` and
`payload.state=interrupted`  
And the resulting interruption preserves
`run.interrupted.interrupted_by=app_server.run.stop`.

### Scenario SCN-0008: Reconnects and resumes run subscribe after missed heartbeat on WebSocket without gaps

Given one non-terminal run exists in the configured app-server run directory  
And one WebSocket client has already completed `initialize`, `initialized`, and
fresh `run/subscribe` for that run  
When the client misses the idle `server/heartbeat` window and treats the
connection as stale  
And the client reconnects, reinitializes, and reissues `run/subscribe` with
`afterSeq` equal to the highest applied canonical event sequence  
Then the server resumes canonical live delivery with only events having
`seq > afterSeq` and without duplicate or missing canonical event sequences.

### Scenario SCN-0009: Serves fresh runs subscribe snapshot after initialize handshake on stdio

Given persisted app-server runs exist in the configured run directory  
When a client completes `initialize` and `initialized` on the stdio transport  
And the client requests `runs/subscribe`  
Then the server returns `payload.items` as one authoritative newest-first run
summary snapshot for the configured app-server run corpus.

### Scenario SCN-0010: Delivers live runs changed upserts for new or changed runs after initialize handshake on stdio

Given one non-terminal run exists in the configured app-server run directory  
When a client completes `initialize` and `initialized` on the stdio transport  
And the client requests `runs/subscribe`  
Then the client receives `runs/changed` notifications with
`payload.kind=upsert` as that run summary
changes.

### Scenario SCN-0011: Delivers live runs changed removals when a persisted run disappears after subscribe

Given one persisted run exists in the configured app-server run directory  
When a client completes `initialize` and `initialized` on the stdio transport  
And the client requests `runs/subscribe`  
And that persisted run directory is removed  
Then the client receives one `runs/changed` notification with
`payload.kind=remove` for that `runId`.

### Scenario SCN-0012: Unsubscribes runs subscribe idempotently after initialize handshake on stdio

Given persisted app-server runs exist in the configured run directory  
When a client completes `initialize` and `initialized` on the stdio transport  
And the client requests `runs/subscribe`  
And the client requests `runs/unsubscribe` twice  
Then the first response reports `payload.unsubscribed=true`  
And the second response reports `payload.unsubscribed=false`.

### Scenario SCN-0013: Replaces duplicate runs subscribe requests without duplicate live delivery after initialize handshake on stdio

Given one non-terminal run exists in the configured app-server run directory  
When a client completes `initialize` and `initialized` on the stdio transport  
And the client requests `runs/subscribe` twice on the same connection  
Then the server replaces the earlier instance-level subscription  
And the client still receives one `runs/changed` delivery for each changed run
summary.

### Scenario SCN-0014: Reconnects and resubscribes runs subscribe after missed heartbeat on WebSocket with fresh authoritative snapshot

Given one non-terminal run exists in the configured app-server run directory  
And one WebSocket client has already completed `initialize`, `initialized`, and
fresh `runs/subscribe`  
When the client misses the idle `server/heartbeat` window and treats the
connection as stale  
And the client reconnects, reinitializes, and reissues `runs/subscribe`  
Then the server returns a fresh authoritative `payload.items` snapshot  
And subsequent `runs/changed` delivery reflects later run-summary changes
without replaying missed intermediate updates.
