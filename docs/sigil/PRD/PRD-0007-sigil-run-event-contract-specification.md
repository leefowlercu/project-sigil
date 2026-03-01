# PRD-0007: Sigil Run Event Contract Specification

## Status

Accepted

## Context

`sigil` requires a deterministic event contract for runtime durability,
replayability, and integrity validation in the event-sourced baseline.

This PRD defines the write-path event envelope, event type catalog, payload
schemas, and per-run durable JSONL storage contract. Query/projection API
behavior is deferred.

## Goals

- Define the required event envelope for v1.
- Define canonical core lifecycle event types for v1.
- Define strict per-event payload schemas and invariants.
- Define per-run sequence ordering guarantees.
- Define durable JSONL storage path/layout.
- Define write durability and integrity failure behavior.

## Non-Goals

- Defining query/read APIs or app-server streaming contracts.
- Defining projection materialization contracts.
- Defining alternative storage backends in v1.
- Defining detailed cross-version migration mechanics.

## Event Envelope Contract

Each event envelope MUST include:

- `event_id` (UUIDv7)
- `schema_version` (string, initial value `v1`)
- `run_id` (UUIDv7)
- `seq` (integer, contiguous, starts at `1` per run)
- `ts` (RFC3339Nano UTC)
- `type` (string enum namespace)
- `node_id` (required for node-scoped events, omitted or `null` for run-scoped events)
- `causation_id` (UUIDv7, required)
- `correlation_id` (UUIDv7, required)
- `payload` (object)

Envelope invariants:

- `correlation_id` MUST equal `run_id` in v1.
- For first run event (`run.queued`), `causation_id` MUST equal its own `event_id`.
- For all subsequent events, `causation_id` MUST reference an existing event in
  the same run with lower `seq`.
- `run.*` events: `node_id` MUST be `null` or omitted.
- `node.*` events: `node_id` MUST be present and UUIDv7.

## Event Type Catalog

Canonical v1 core lifecycle event types are:

- `run.queued`
- `run.running`
- `node.started`
- `node.completed`
- `run.completed`
- `run.failed`
- `run.interrupted`

No additional event types are accepted in v1 validation.

## Payload Schema by Event Type

All payload schemas in this section are strict:

- Unknown payload fields MUST fail validation.
- Additional payload properties are forbidden unless explicitly listed.

### `run.queued` payload schema

| Field | Required | Type | Invariants |
| --- | --- | --- | --- |
| `source` | Yes | string enum | One of: `cli.run.start`, `app_server.run.start`, `internal.resume` |
| `app_config_path` | No | string | Non-empty when present |
| `run_config_path` | No | string | Non-empty when present |

Additional invariants:

- `app_config_path` and `run_config_path` MAY be omitted in v1.
- For file-based run initiation sources, corresponding config path fields SHOULD
  be present.

### `run.running` payload schema

| Field | Required | Type | Invariants |
| --- | --- | --- | --- |
| `executor` | Yes | string enum | Must be `rlm` |
| `max_depth` | Yes | integer | `>= 0` |

### `node.started` payload schema

| Field | Required | Type | Invariants |
| --- | --- | --- | --- |
| `depth` | Yes | integer | `>= 0` |
| `parent_node_id` | Yes | UUIDv7 or `null` | `null` when root |
| `role` | Yes | string enum | One of: `root`, `recursive_subcall` |
| `attempt` | Yes | integer | `>= 1` |

Additional invariants:

- `depth == 0` => `parent_node_id == null` and `role == root`
- `depth > 0` => `parent_node_id` UUIDv7 and `role == recursive_subcall`

### `node.completed` payload schema

| Field | Required | Type | Invariants |
| --- | --- | --- | --- |
| `status` | Yes | string literal | Must be `completed` |
| `duration_ms` | Yes | integer | `>= 0` |
| `output_ref` | No | string | Non-empty when present; SHOULD reference persisted normalized inference output with `schema_id=sigil.rlm.response.v1` |

### `run.completed` payload schema

| Field | Required | Type | Invariants |
| --- | --- | --- | --- |
| `status` | Yes | string literal | Must be `completed` |
| `duration_ms` | Yes | integer | `>= 0` |
| `final_answer_ref` | No | string | Non-empty when present; SHOULD reference terminal normalized inference output with `schema_id=sigil.rlm.response.v1` and `validated_payload.decision=final` |

### `run.failed` payload schema

| Field | Required | Type | Invariants |
| --- | --- | --- | --- |
| `status` | Yes | string literal | Must be `failed` |
| `error_code` | Yes | string | Non-empty |
| `error_message` | Yes | string | Non-empty |
| `failed_node_id` | No | UUIDv7 | Optional |
| `retryable` | Yes | boolean | None |

### `run.interrupted` payload schema

| Field | Required | Type | Invariants |
| --- | --- | --- | --- |
| `status` | Yes | string literal | Must be `interrupted` |
| `reason` | Yes | string enum | One of: `user_request`, `policy_stop`, `system_shutdown` |
| `interrupted_by` | No | string | Non-empty when present |
| `interrupted_node_id` | No | UUIDv7 | Optional |

## Extensibility Rules

v1 uses strict validation rules:

- Unknown envelope fields MUST fail validation.
- Unknown payload fields MUST fail validation.
- Unknown event types MUST fail validation.
- Only `schema_version == v1` is valid for this baseline.
- Exact cross-version forward/backward compatibility mechanics are deferred.
- Until cross-version mechanics are specified, exact-version matching is
  required.

## Event Ordering Contract

- Per-run `seq` MUST be contiguous monotonic integers beginning at `1`.
- Append operation MUST reject an event when next `seq` is not contiguous.
- Sequence ordering is authoritative for replay order within a run.

## Durable Storage Contract

- Durable run store base path MUST be `./sigil/runs`.
- Per-run directory MUST be `./sigil/runs/<run_id>/`.
- Event log file MUST be `events.jsonl`.
- Event file format MUST be one JSON object per line.
- Event store MUST be append-only; in-place mutation/rewrite is forbidden.

## Write Durability and Integrity Contract

- Writer MUST `fsync` every appended event record before acknowledging
  persistence.
- Recovery MUST treat any of the following as integrity failure:
  - sequence gap
  - malformed JSON line
  - partial/truncated line
- Integrity failure handling MUST fail validation for run recovery until repaired
  by explicit operator or tooling action.

## Deferred Payloads

The following payload families are explicitly deferred (not locked in this PRD):

- Tool invocation internals (arguments/results traces).
- Model provider request/response internals (token-level or raw provider
  payloads).
- Code-execution internals (stdout/stderr chunks, sandbox telemetry).
- Provider usage/cost telemetry internals.

Deferred payload families are out-of-contract for v1 and MUST NOT be introduced
into core lifecycle payload schemas without a follow-up PRD update.

## Deferred Contracts

The following are explicitly deferred to future PRDs:

- Query/read/streaming API behavior.
- Projection storage contracts.
- Storage backend alternatives beyond JSONL baseline.
- Cross-version migration and compatibility mechanics.

## Acceptance Scenarios

### Scenario SCN-0000: Persists run events to per-run append-only events.jsonl under sigil runs directory

Given a run emits runtime events  
When events are persisted  
Then events are appended to `./sigil/runs/<run_id>/events.jsonl`.

### Scenario SCN-0001: Uses UUIDv7 identifiers for run node and event identity fields

Given a persisted event record  
When identity fields are inspected  
Then `run_id`, `node_id` (when present), and `event_id` are UUIDv7 values.

### Scenario SCN-0002: Assigns contiguous per-run sequence numbers starting at one

Given persisted events for a run  
When sequence values are inspected in append order  
Then `seq` starts at `1` and increments contiguously by `1`.

### Scenario SCN-0003: Writes one valid JSON event envelope per line

Given persisted run events  
When `events.jsonl` is parsed line-by-line  
Then each non-empty line is a valid JSON event envelope.

### Scenario SCN-0004: Requires run_id on all events and node_id on node-scoped events

Given persisted run-scoped and node-scoped events  
When required fields are validated  
Then all events contain `run_id` and node-scoped events contain `node_id`.

### Scenario SCN-0005: Fsyncs each appended event before acknowledging persistence

Given an event append operation  
When persistence acknowledgement is returned  
Then the event record has been fsynced to durable storage.

### Scenario SCN-0006: Rejects event append when next sequence is not contiguous

Given a run with latest persisted sequence `N`  
When an append is requested with `seq` not equal to `N+1`  
Then append fails validation and the event is not persisted.

### Scenario SCN-0007: Fails integrity validation on malformed or partial persisted event lines

Given `events.jsonl` containing malformed JSON or partial lines  
When run recovery integrity validation executes  
Then validation fails with integrity error.

### Scenario SCN-0008: Preserves immutability by forbidding in-place event modification

Given persisted events for a run  
When a mutation/rewrite of existing event lines is attempted  
Then operation is rejected by event-store contract.

### Scenario SCN-0009: Carries schema_version to support forward event evolution

Given persisted v1 events  
When envelopes are inspected  
Then `schema_version` exists and equals `v1` for baseline records.

### Scenario SCN-0010: Defines canonical core lifecycle event type catalog for v1

Given v1 run-event validation rules  
When event type is validated  
Then only the canonical core lifecycle event types are accepted.

### Scenario SCN-0011: Enforces strict payload schema and invariants for each core lifecycle event type

Given persisted events for canonical v1 lifecycle types  
When payloads are validated  
Then required fields, field types, and event-specific invariants are enforced.

### Scenario SCN-0012: Rejects unknown fields and unknown event types under v1 strict extensibility rules

Given an event with unknown envelope fields, unknown payload fields, or unknown
`type`  
When v1 validation executes  
Then validation fails and the event is rejected.

### Scenario SCN-0013: Defers non-core tool and model payload families while keeping core lifecycle payloads normative

Given payload fields from deferred non-core families  
When validating against core v1 lifecycle payload schemas  
Then those fields are out-of-contract and rejected until a follow-up PRD update.
