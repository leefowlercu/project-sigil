# PRD-0180: Sigil App-Server Read Query Surface Specification

## Status

Accepted

## Context

The app-server read plane needs richer run-oriented queries than the existing
CLI surfaces, but it must still share one underlying query implementation with
`sigil run list`, `sigil run status`, `sigil run inspect`, and
`sigil run events`.

The implemented app-server read plane now exposes:

- `runs/list`
- `run/read`
- `run/events/read`
- `run/tree/read`
- `run/steps/list`
- `run/node/read`
- `run/step/read`
- `run/artifact/read`

## Goals

- Define the v1 JSON-RPC read-method families and response envelopes.
- Keep CLI and app-server query semantics aligned through one shared
  `internal/query` package.
- Expose tree, node, step, and artifact drill-in payloads without forcing rich
  clients to reconstruct those views solely from raw event replay.

## Non-Goals

- Defining live subscriptions or run execution control.
- Defining a second app-server-only query implementation.
- Defining remote filters beyond `runs/list` limit and cursor support.

## Read Contract

- `runs/list` MUST return paged run summaries ordered newest-first.
- `runs/list` MUST accept `limit` and `cursor` parameters from the first
  implementation slice.
- `run/read` MUST return one run-scoped projection payload without inlining
  large artifact bodies.
- `run/events/read` MUST return canonical persisted event history for one run.
- `run/tree/read` MUST return explicit `childNodeIds` linkage in canonical
  node-start order.
- `run/steps/list` MUST return a run-global step timeline ordered by canonical
  `node.step.started.seq`.
- `run/node/read` MUST include `stepIds`, `activeStepId`, and `childNodeIds`.
- `run/step/read` MUST include turn refs, action refs,
  `subcallAccountingRefs`, accounting ref, and explicit step state.
- `run/artifact/read` MUST resolve canonical artifact refs and return:
  - `artifactRef`
  - `artifactKind`
  - sparse `identity`
  - typed JSON `artifact`

## Shared Query Requirement

- App-server read methods MUST be implemented through one shared query package
  that is also used by CLI `run list`, `run status`, `run inspect`, and
  `run events`.
- This contract MUST NOT introduce a second app-server-only query stack, even
  briefly.
- CLI output shaping MAY remain CLI-specific, but data loading and projection
  logic MUST stay shared.

## Error Contract

- Malformed identifiers, invalid cursors, and invalid artifact refs MUST
  return standard invalid-params JSON-RPC codes with stable domain codes under
  `error.data.code`.
- Missing run, node, step, and artifact targets MUST return stable domain
  codes:
  - `run_not_found`
  - `node_not_found`
  - `step_not_found`
  - `artifact_not_found`

## Acceptance Scenarios

### Scenario SCN-0000: Serves paged runs list after initialize handshake on stdio

Given persisted runs exist in the configured app-server run directory  
When a client completes `initialize` and `initialized` on the stdio transport  
And the client requests `runs/list` with `limit=1`  
Then the server returns one run item plus a stable `nextCursor`.

### Scenario SCN-0001: Serves run projection and canonical event history after initialize handshake on stdio

Given one persisted run exists in the configured app-server run directory  
When a client completes `initialize` and `initialized` on the stdio transport  
And the client requests `run/read` for that run  
Then the server returns one run-scoped projection payload with `runId`,
`asOfSeq`, and `payload.run`  
When the client requests `run/events/read` for the same run  
Then the server returns canonical persisted event history for that run.

### Scenario SCN-0002: Serves node tree and node linkage after initialize handshake on stdio

Given one persisted run includes a root node, a child node, and one completed
step  
When a client completes `initialize` and `initialized` on the stdio transport  
And the client requests `run/tree/read` for that run  
Then the server returns `rootNodeId`, node ordering, and explicit
`childNodeIds` linkage  
When the client requests `run/node/read` for the root node  
Then the server returns `stepIds` and `childNodeIds` for that node.

### Scenario SCN-0003: Serves run-global step timeline after initialize handshake on stdio

Given one persisted run includes one completed step with canonical step metadata  
When a client completes `initialize` and `initialized` on the stdio transport  
And the client requests `run/steps/list` for that run  
Then the server returns one run-global step timeline ordered by
`node.step.started.seq`.

### Scenario SCN-0004: Serves step detail and typed artifact bodies after initialize handshake on stdio

Given one persisted run includes a completed step with one action artifact  
When a client completes `initialize` and `initialized` on the stdio transport  
And the client requests `run/step/read` for that step  
Then the server returns step refs and stable step metadata  
When the client requests `run/artifact/read` for the action artifact  
Then the server returns `artifactKind`, sparse identity metadata, and the
typed artifact JSON body.
