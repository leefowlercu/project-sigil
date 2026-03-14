# PRD-0140: Sigil Application Logging Specification

## Status

Accepted

## Context

`sigil` requires deterministic application logging for operational debugging and
startup verification. Logging is now configured through the grouped
application-config contract owned by `PRD-0100`.

## Goals

- Define baseline application log output format.
- Define how effective log file paths are derived from `logs.dir`.
- Define startup behavior when logging sink initialization fails.

## Non-Goals

- Defining log rotation or retention behavior.
- Defining multi-sink logging behavior.
- Defining app-server notification or runtime-event logging semantics.

## Logging Output Contract

- Application logs MUST be persisted to file only in the baseline contract.
- Application logs MUST use structured JSON records.
- Structured JSON records MUST be based on baseline `slog` JSON handler
  behavior.

## Log File Path Derivation Contract

- The log file name MUST be `sigil.log`.
- The effective application log file path MUST be derived as
  `<logs.dir>/sigil.log`.
- If `logs.dir` is relative, path resolution MUST follow `PRD-0100` rules.
- With default `PRD-0100` values, the default log file path MUST be
  `./.sigil/logs/sigil.log`.

## Initialization Failure Contract

- If the derived log file path cannot be opened or created as a file sink,
  application startup MUST fail with non-zero exit.
- Missing, unusable, or non-file sink targets MUST be treated as logging
  initialization failure.

## Deferred Contracts

The following are explicitly deferred to future PRDs:

- Log rotation policy and retention windows.
- Additional sinks such as mirrored stderr.
- Pinned JSON field-schema guarantees beyond baseline structured output.

## Acceptance Scenarios

### Scenario SCN-0000: Writes application logs to a derived sigil.log file path

Given an effective `logs.dir` value  
When application logging is initialized  
Then the effective log file path is `<logs.dir>/sigil.log`.

### Scenario SCN-0001: Uses JSON structured log records for application logging

Given application logging emits records  
When log entries are written  
Then records are emitted in structured JSON format.

### Scenario SCN-0002: Uses default log file path when default logs.dir is in effect

Given default `PRD-0100` application configuration values  
When logging is initialized  
Then the effective log target path is `./.sigil/logs/sigil.log`.

### Scenario SCN-0003: Fails initialization when derived log file path cannot be opened as a file sink

Given a derived log file path that cannot be opened or created as a file sink  
When logging initialization runs  
Then startup fails with non-zero exit.

### Scenario SCN-0004: Derives log file path from configured logs.dir override

Given an overridden `logs.dir` value  
When logging is initialized  
Then the effective log target path is `<overridden-logs.dir>/sigil.log`.
