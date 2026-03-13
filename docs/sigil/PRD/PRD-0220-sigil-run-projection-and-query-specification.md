# PRD-0220: Sigil Run Projection and Query Specification

## Status

Draft

## Context

`sigil` already persists canonical lifecycle events and auxiliary local control
metadata for each run, but operators currently have to interpret those files
directly.

The inspection command surface needs a stable read-model contract that derives
run summaries, detailed projections, and canonical event queries without
creating a second persisted source of truth.

## Goals

- Define the authoritative inputs for run summary and run projection queries.
- Define the derived field set exposed by run summaries and detailed run
  projections.
- Define how canonical event queries are read and validated.
- Keep read-model behavior reusable for future CLI and app-server surfaces.

## Non-Goals

- Defining persisted materialized views or cached projections.
- Defining remote query transport or streaming APIs.
- Defining trajectory visualization UX beyond the derived projection payload.

## Query Source Contract

- Run summary and run projection queries MUST derive from validated canonical
  events stored in `events.jsonl`.
- Query implementations MAY consult auxiliary local control metadata from
  `process.json` and `stop-request.json` when present.
- Query implementations MUST treat auxiliary control metadata as derived
  inspection inputs and MUST NOT let those files supersede canonical event-log
  state.
- Query implementations MAY surface output refs, accounting refs, and artifact
  refs already recorded by canonical events or run-local outputs.
- Query implementations MUST NOT require inline expansion of large output or
  artifact bodies in order to satisfy the run projection contract.
- Query implementations MUST derive results on demand and MUST NOT persist a
  separate materialized read model in v1.

## Run Summary Contract

- A run summary MUST include:
  - `run_id`
  - `state`
  - `source`
  - `queued_at`
  - `started_at` when `run.running` was observed
  - `terminal_at` when a terminal run event was observed
  - `events_path`
  - `pid_status`
  - `stop_requested`
  - `final_answer_ref` when present
  - `accounting_ref` when present
- `final_answer_ref` and `accounting_ref` SHOULD be exposed as canonical
  artifact refs rather than inline terminal artifact bodies.
- `pid_status` MUST be one of:
  - `current`
  - `missing`
  - `not_running`
  - `stale`
- `pid_status=current` means `process.json` is present and still identifies the
  original live process for the run.
- `pid_status=missing` means no usable `process.json` exists for the run.
- `pid_status=not_running` means `process.json` exists but the recorded process
  no longer exists.
- `pid_status=stale` means `process.json` exists but no longer identifies the
  original process for that run.
- `stop_requested=true` means `stop-request.json` exists and validates for that
  run.

## Run Projection Contract

- A run projection MUST include the full run summary.
- A run projection MUST include resolved application and run config-path
  metadata from `run.queued` when present.
- A run projection MUST include executor and max-depth metadata from
  `run.running` when present.
- A run projection MUST include terminal payload details appropriate to the
  observed terminal state:
  - `final_answer_ref` and `accounting_ref` for completed runs
  - failure metadata such as `error_code`, `error_message`, `failed_node_id`,
    and `failed_step_id` for failed runs
  - interruption metadata such as `reason`, `interrupted_by`, and
    `interrupted_node_id` for interrupted runs
- A run projection MUST include derived counts for nodes, steps, actions, and
  subcalls.
- A run projection MUST include per-node summaries derived from canonical node
  events.
- A run projection MUST preserve output and accounting references as references
  rather than replacing them with inline artifact bodies.

## Canonical Event Query Contract

- Canonical event queries MUST return validated event envelopes in persisted
  append order.
- Canonical event queries MUST fail when `events.jsonl` is missing, corrupt, or
  violates canonical validation rules.
- Canonical event queries MUST preserve the canonical event envelope field set
  defined by `PRD-0210-sigil-run-event-contract-specification.md`.

## Error Contract

- Targeted run summary and run projection queries MUST fail with non-zero exit
  behavior when canonical event logs are missing or corrupt for the targeted
  run.
- Query surfaces MUST NOT silently repair invalid canonical event content.

## Deferred Contracts

The following are explicitly deferred to future PRDs:

- Persisted projection caches or indexing structures.
- Query filtering and paging semantics.
- Remote projection APIs and streaming subscriptions.

## Acceptance Scenarios

### Scenario SCN-0000: Derives run summary from canonical events and auxiliary control metadata

Given canonical events and auxiliary control metadata exist for one run  
When a run summary is requested  
Then the summary is derived from canonical events plus auxiliary control
metadata.

### Scenario SCN-0001: Derives run projection from canonical events without materializing a second source of truth

Given canonical events and run-local refs exist for one run  
When a run projection is requested  
Then the projection is derived on demand without persisting a separate read
model.

### Scenario SCN-0002: Reports pid status as current missing not_running or stale from process metadata

Given process metadata states vary across runs  
When run summaries are derived  
Then `pid_status` reports `current`, `missing`, `not_running`, or `stale`
accordingly.

### Scenario SCN-0003: Surfaces stop_requested from stop-request metadata without changing event authority

Given stop-request metadata exists for one run  
When a run summary or run projection is requested  
Then `stop_requested` is surfaced without changing canonical event authority.

### Scenario SCN-0004: Preserves final answer and accounting as refs in run projection output

Given canonical terminal refs exist for one run  
When a run projection is requested  
Then final-answer and accounting data are exposed as refs rather than inline
artifact bodies.

### Scenario SCN-0005: Fails targeted run queries when canonical event logs are missing or corrupt

Given a targeted run has missing or corrupt canonical event storage  
When a targeted run query is requested  
Then the query fails rather than returning a repaired projection.
