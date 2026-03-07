# PRD-0200: Sigil Run Lifecycle State Machine Specification

## Status

Accepted

## Context

`sigil` needs a deterministic lifecycle contract for recursive run execution so
runtime behavior can be validated independently from CLI UX placeholders.

This PRD defines run and node lifecycle semantics for the runtime write-path
baseline and intentionally does not supersede existing CLI placeholder behavior
contracts.

## Goals

- Define run lifecycle states and allowed transitions.
- Define root/child node lifecycle invariants for recursive execution.
- Define interruption and failure state behavior.
- Provide acceptance scenarios for lifecycle correctness.

## Non-Goals

- Defining app-server read/query/streaming APIs.
- Defining projection/query contracts.
- Changing CLI UX contracts in `PRD-0110` or `PRD-0130`.

## Run Lifecycle States

Run states in v1 MUST be:

- `queued`
- `running`
- `completed`
- `failed`
- `interrupted`

Initial state MUST be `queued`.

Allowed transitions:

- `queued -> running`
- `running -> completed`
- `running -> failed`
- `running -> interrupted`

Terminal states are:

- `completed`
- `failed`
- `interrupted`

Terminal run states MUST reject further transitions.

## Node Lifecycle and Tree Invariants

- One root node MUST be started when run enters `running`.
- Root node invariants:
  - `depth=0`
  - `parent_node_id=null`
- Child nodes MAY be created only while run is `running`.
- Every child node MUST reference an existing parent node in the same run.
- Tool/code-exec activity MUST remain node-scoped event activity and MUST NOT
  create additional node entities in v1.

## Transition Rules and Validation

- Transition requests MUST be validated against allowed transition graph.
- Any disallowed transition MUST fail validation.
- Transitions from terminal states MUST be rejected.
- Node creation outside `running` state MUST be rejected.
- Child node creation without valid same-run parent MUST be rejected.

## Failure/Interruption Behavior

- Unrecoverable runtime failure MUST transition run to `failed`.
- Explicit interruption MUST transition run to `interrupted`.
- Once interrupted, failed, or completed, run state remains terminal.

## Deferred Contracts

The following are explicitly deferred to future PRDs:

- App-server control/query API contracts.
- Detailed retry policy semantics beyond node-scoped attempt metadata.
- Projection/read-model contracts.

## Acceptance Scenarios

### Scenario SCN-0000: Initializes runs in queued state before execution begins

Given a new run is created  
When lifecycle initialization completes  
Then run state is `queued`.

### Scenario SCN-0001: Transitions run from queued to running when execution starts

Given a run in `queued` state  
When execution starts  
Then run transitions to `running`.

### Scenario SCN-0002: Creates exactly one root node at depth zero for each run

Given a run transitions to `running`  
When node initialization occurs  
Then exactly one root node exists with `depth=0` and `parent_node_id=null`.

### Scenario SCN-0003: Allows child nodes only under an existing parent node in the same run

Given a run in `running` state  
When a child node is created  
Then it references an existing parent node in the same run.

### Scenario SCN-0004: Transitions run to completed on successful execution termination

Given a run in `running` state  
When execution terminates successfully  
Then run transitions to `completed`.

### Scenario SCN-0005: Transitions run to failed on unrecoverable runtime failure

Given a run in `running` state  
When unrecoverable runtime failure occurs  
Then run transitions to `failed`.

### Scenario SCN-0006: Transitions run to interrupted on explicit interruption

Given a run in `running` state  
When explicit interruption is requested  
Then run transitions to `interrupted`.

### Scenario SCN-0007: Rejects invalid transitions from terminal run states

Given a run in a terminal state (`completed`, `failed`, or `interrupted`)  
When any further state transition is requested  
Then transition validation fails.

### Scenario SCN-0008: Represents tool and code execution as node-scoped events without creating nodes

Given a run in `running` state with active recursive nodes  
When tool or code execution activity occurs  
Then activity is recorded as node-scoped events and no additional node entity is created.
