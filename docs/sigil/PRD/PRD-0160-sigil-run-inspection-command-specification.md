# PRD-0160: Sigil Run Inspection Command Specification

## Status

Draft

## Context

`sigil` already persists canonical run events, artifact refs, accounting refs,
and local control metadata under the run storage directory.

Operators need a first-class CLI surface for listing runs, checking one run's
status, inspecting one run's derived summary, and reading canonical events
without manually reconstructing state from the filesystem.

## Goals

- Define `sigil run list`, `sigil run status`, `sigil run inspect`, and
  `sigil run events`.
- Define inherited `--run-dir` and `--output` behavior for inspection commands.
- Define text and JSON output expectations for inspection commands.
- Define best-effort listing behavior and strict targeted-query behavior.

## Non-Goals

- Defining app-server or remote read APIs.
- Defining live watch, tail, or follow semantics.
- Defining projection field semantics beyond the runtime read-model ownership in
  `PRD-0220-sigil-run-projection-and-query-specification.md`.

## Command Input Contract

- `sigil run list` MUST be invoked as `sigil run list` with no positional
  arguments.
- `sigil run status` MUST be invoked as `sigil run status <run-id>`.
- `sigil run inspect` MUST be invoked as `sigil run inspect <run-id>`.
- `sigil run events` MUST be invoked as `sigil run events <run-id>`.
- All inspection commands MUST accept inherited `-o, --output <text|json>`
  values from the root CLI surface defined in `PRD-0110`.
- All inspection commands MUST accept inherited `--run-dir <path>` values from
  the `sigil run` parent command.
- When `--run-dir` is omitted, inspection commands MUST target the default
  storage base directory `./.sigil/runs`.
- For `status`, `inspect`, and `events`, `run-id` MUST be a positional
  argument, not a flag.
- `status`, `inspect`, and `events` MUST require exactly one positional
  `run-id` argument.
- The provided `run-id` for `status`, `inspect`, and `events` MUST validate as
  UUIDv7 before any filesystem lookup occurs.
- No inspection-specific flags beyond inherited `--output` and `--run-dir` are
  introduced in v1.

## Output Contract

- `sigil run list` text output MUST render one human-readable summary entry per
  discovered run.
- `sigil run list --output json` MUST render one JSON array of run summaries.
- `sigil run list` MUST sort entries newest-first using queued-time ordering.
- `sigil run list` MUST return success with an empty result when the selected
  run directory does not exist yet.
- `sigil run list` MUST continue past corrupt or unreadable discovered run
  entries and surface a per-entry error summary instead of failing the entire
  command.
- `sigil run status` text output MUST render one compact human-readable run
  summary.
- `sigil run status --output json` MUST render one JSON object using the run
  summary contract owned by `PRD-0220`.
- `sigil run inspect` text output MUST render one detailed human-readable run
  inspection summary.
- `sigil run inspect --output json` MUST render one JSON object using the run
  projection contract owned by `PRD-0220`.
- `sigil run events` text output MUST render compact validated canonical event
  lines in append order.
- `sigil run events --output json` MUST render one JSON array of canonical
  event envelopes in append order.

## Error Contract

- `sigil run status`, `sigil run inspect`, and `sigil run events` MUST fail
  with non-zero exit when the targeted run is unknown.
- `sigil run status`, `sigil run inspect`, and `sigil run events` MUST fail
  with non-zero exit when the targeted run event log is corrupt or invalid.
- Invalid inherited `--output` values MUST fail command validation before any
  inspection lookup begins.
- Unknown or unsupported flags SHOULD follow Cobra default behavior and SHOULD
  exit non-zero.

## Deferred Contracts

The following are explicitly deferred to future PRDs:

- Watch or follow semantics for event inspection.
- Filtering, paging, or server-side query parameters.
- Remote or app-server inspection transports.

## Acceptance Scenarios

### Scenario SCN-0000: Lists runs newest-first from the selected run directory

Given one or more persisted runs exist in the selected run directory  
When a user runs `sigil run list`  
Then the returned run summaries are ordered newest-first by queued time.

### Scenario SCN-0001: Returns empty success when sigil run list targets a missing selected run directory

Given the selected run directory does not exist  
When a user runs `sigil run list`  
Then the command exits with status code `0` and returns an empty result.

### Scenario SCN-0002: Requires exactly one UUIDv7 run-id positional argument for sigil run status

Given `sigil run status` command inputs are evaluated  
When the command is invoked with missing extra or non-UUIDv7 `run-id` input  
Then command validation fails with non-zero exit.

### Scenario SCN-0003: Requires exactly one UUIDv7 run-id positional argument for sigil run inspect

Given `sigil run inspect` command inputs are evaluated  
When the command is invoked with missing extra or non-UUIDv7 `run-id` input  
Then command validation fails with non-zero exit.

### Scenario SCN-0004: Requires exactly one UUIDv7 run-id positional argument for sigil run events

Given `sigil run events` command inputs are evaluated  
When the command is invoked with missing extra or non-UUIDv7 `run-id` input  
Then command validation fails with non-zero exit.

### Scenario SCN-0005: Rejects invalid inherited output value for sigil run inspection commands

Given a run inspection command receives an inherited `--output` value outside
`text|json`  
When command input validation runs  
Then command validation fails with non-zero exit before any inspection lookup
occurs.

### Scenario SCN-0006: Prints run list summaries in text and json output modes

Given persisted runs exist in the selected run directory  
When a user runs `sigil run list` in text mode and in json mode  
Then both outputs return the same run-summary set in their respective formats.

### Scenario SCN-0007: Prints run status summary in text and json output modes

Given a targeted persisted run exists in the selected run directory  
When a user runs `sigil run status <run-id>` in text mode and in json mode  
Then both outputs return the same run summary in their respective formats.

### Scenario SCN-0008: Prints run inspection summary in text and json output modes

Given a targeted persisted run exists in the selected run directory  
When a user runs `sigil run inspect <run-id>` in text mode and in json mode  
Then both outputs return the same run inspection summary in their respective
formats.

### Scenario SCN-0009: Returns canonical run events in append order for sigil run events

Given a targeted persisted run exists in the selected run directory  
When a user runs `sigil run events <run-id>`  
Then the returned event stream preserves canonical append order.

### Scenario SCN-0010: Uses inherited run-dir override across sigil run inspection commands

Given persisted runs exist outside the default run storage directory  
When a user runs a run inspection command with inherited `--run-dir <path>`  
Then the command inspects only the runs stored under `<path>`.
