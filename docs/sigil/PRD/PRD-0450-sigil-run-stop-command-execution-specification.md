# PRD-0450: Sigil Run Stop Command Execution Specification

## Status

Draft

## Context

`sigil run start` is a blocking local CLI process, but the runtime already owns
durable run state, interruption events, and partial accounting semantics.

`sigil run stop` needs a first-class execution contract that can gracefully
stop an in-progress local CLI run without introducing app-server control APIs in
v1.

## Goals

- Define `sigil run stop` as a local graceful-stop command for CLI-started runs.
- Define the v1 PID-and-signal control transport and auxiliary control files.
- Define wait-to-terminal behavior and success output semantics.
- Define user-stop interruption behavior for active and startup-window runs.

## Non-Goals

- Defining app-server or remote run-control APIs.
- Defining run discovery or list/query commands.
- Defining non-Unix transport behavior in v1.

## Command Runtime Contract

- `sigil run stop` MUST target local runs stored under `./.sigil/runs/<run-id>/`.
- The event log at `./.sigil/runs/<run-id>/events.jsonl` MUST remain the source
  of truth for run state.
- `sigil run stop` MUST load and validate `events.jsonl` before deciding whether
  a stop request is needed.
- If the run is already terminal (`completed`, `failed`, or `interrupted`), the
  command MUST return success without issuing a new stop request.
- If the run is non-terminal, the command MUST write
  `./.sigil/runs/<run-id>/stop-request.json` before sending `SIGTERM` to the
  active process.
- Before signaling a non-terminal run, `sigil run stop` MUST validate that
  `process.json` still identifies the original live process for that run.
- After sending `SIGTERM`, the command MUST wait until a terminal run state is
  observed in `events.jsonl`.

## Output Contract

- Successful command output MUST be one JSON object including:
  - `run_id`
  - `stop_requested`
  - `state`
  - `events_path`
- `stop_requested` MUST be `false` only when the command observed a terminal
  run before issuing a new stop request.
- `state` MUST report the final terminal state observed by the command.
- If a stop request loses the race to `completed` or `failed`, the command MUST
  still exit `0` and return the observed terminal state with
  `stop_requested=true`.

## Control Transport and Metadata Contract

- v1 stop transport MUST be Unix-like only and use `SIGTERM`.
- `sigil run start` MUST publish `./.sigil/runs/<run_id>/process.json` once the
  run is created, with:
  - `run_id`
  - `pid`
  - `recorded_at`
  - `started_at`
  - `source=cli.run.start`
- `started_at` MUST identify the OS-observed start time for the recorded PID so
  stop commands can reject stale or reused PID metadata safely.
- `stop-request.json` MUST include:
  - `run_id`
  - `requested_at`
  - `requested_by=cli.run.stop`
  - `signal=SIGTERM`
- `process.json` is auxiliary control metadata and MUST NOT supersede
  `events.jsonl` as state authority.
- `process.json` MUST be removed when the local CLI process exits normally.

## Interruption Semantics

- `sigil run start` MUST trap `SIGTERM` and convert it into graceful
  interruption rather than immediate process termination.
- A user stop MUST terminalize the run as `run.interrupted`, never `run.failed`.
- When `stop-request.json` is present, the resulting `run.interrupted` payload
  MUST use:
  - `reason=user_request`
  - `interrupted_by=cli.run.stop`
- `run.interrupted.interrupted_node_id` SHOULD be set when the active node is
  known at interruption time.
- Active work interrupted by user stop MUST NOT append synthetic `node.failed`
  or `node.step.completed` records solely because cancellation occurred.
- Interrupted runs MUST include partial accounting and optional `accounting_ref`
  as defined by `PRD-0510`.

## Startup-Window Contract

- `sigil run stop` MUST remain deterministic when a stop request arrives after
  `run.queued` is persisted but before `run.running` is persisted.
- `process.json` MUST be published before `run.queued` becomes observable to
  external stop commands.
- In that startup window, the local CLI process MUST still terminalize the run
  as `interrupted`.

## Error Contract

- Unknown `run-id` values MUST fail with non-zero exit.
- Invalid or corrupt `events.jsonl` content MUST fail with non-zero exit.
- A non-terminal run missing usable `process.json` metadata MUST fail with
  non-zero exit.
- A non-terminal run with stale or reused `process.json` identity metadata MUST
  fail with non-zero exit without signaling the mismatched PID.
- This PRD does not require custom error text beyond stable typed behavior and
  non-zero exit.

## Deferred Contracts

The following are explicitly deferred to future PRDs:

- Remote stop transport and app-server control APIs.
- Run discovery or list/query commands.
- Windows or non-signal transport behavior.

## Acceptance Scenarios

### Scenario SCN-0000: Interrupts an actively running CLI run and prints a terminal JSON stop result

Given a local CLI run is actively executing  
When a user runs `sigil run stop <run-id>`  
Then the run transitions to `interrupted`  
And stdout contains one JSON stop result with `run_id`, `stop_requested`,
`state`, and `events_path`.

### Scenario SCN-0001: Publishes process metadata and persists stop-request metadata for local CLI run control

Given a local CLI run is actively executing  
When the run lifecycle and stop request metadata are inspected  
Then `process.json` exists for the active run  
And `stop-request.json` is written before `SIGTERM` is issued.

### Scenario SCN-0002: Returns terminal no-op stop results for completed failed or interrupted runs

Given a local CLI run is already in terminal state  
When a user runs `sigil run stop <run-id>`  
Then the command exits with status code `0`  
And the JSON stop result contains `stop_requested=false`.

### Scenario SCN-0003: Waits for terminal state and reports completed or failed when stop loses the race

Given a stop request is issued for a local CLI run  
When the target run reaches `completed` or `failed` before interruption is persisted  
Then the command exits with status code `0`  
And the JSON stop result contains `stop_requested=true` and the observed
terminal `state`.

### Scenario SCN-0004: Fails run stop for unknown runs corrupt event logs or stale process metadata on non-terminal runs

Given `sigil run stop` targets an unknown run corrupt event log or non-terminal run with missing or stale process metadata  
When the command validates local control state  
Then the command exits non-zero.

### Scenario SCN-0005: Converts startup-window SIGTERM into interrupted terminalization before run.running

Given a local CLI run has persisted `run.queued` but not yet `run.running`  
When `sigil run stop` issues `SIGTERM` for that run  
Then the run terminalizes as `interrupted`.

### Scenario SCN-0006: Persists user-request interruption metadata and partial accounting without synthetic node failure records

Given a local CLI run is interrupted by `sigil run stop` while work is active  
When interruption terminalization is persisted  
Then `run.interrupted` contains `reason=user_request`,
`interrupted_by=cli.run.stop`, and partial accounting  
And the interrupted work does not emit synthetic `node.failed` or
`node.step.completed` records solely because of the stop request.
