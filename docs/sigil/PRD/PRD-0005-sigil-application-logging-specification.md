# PRD-0005: Sigil Application Logging Specification

## Status

Accepted

## Context

`sigil` requires a deterministic baseline logging behavior contract for:

- Structured, machine-readable application logs.
- Stable file-target behavior for operational debugging.
- Predictable startup failure behavior when logging sink initialization fails.

This PRD defines behavior-level logging output and file target contracts.

## Goals

- Define baseline application log output format.
- Define how effective log file paths are derived.
- Define startup/initialization behavior when file sink setup fails.
- Keep contracts aligned with `PRD-0001` application config (`log_dir`,
  `log_level`).

## Non-Goals

- Defining log rotation or retention behavior.
- Defining multi-sink logging behavior (for example file + stderr).
- Defining strict, pinned JSON field schema beyond baseline structured JSON.

## Logging Output Contract

- Application logs MUST be persisted to file only in this baseline.
- Application logs MUST be structured JSON records.
- Structured JSON records are based on baseline `slog` JSON handler behavior.

## Log File Path Derivation Contract

- The log file name MUST be `sigil.log`.
- The effective application log file path MUST be derived as
  `<log_dir>/sigil.log`.
- If `log_dir` is relative, path resolution MUST follow `PRD-0001` rules
  (relative to current working directory).
- With default `PRD-0001` application config values, the default log file path
  MUST be `./sigil/logs/sigil.log`.

## Initialization Failure Contract

- If the derived log file path cannot be opened/created as a file sink,
  application startup/initialization MUST fail with non-zero exit.
- Missing, unusable, or non-file sink targets MUST be treated as initialization
  failure for the logging subsystem.

## Deferred Contracts

The following are explicitly deferred to future PRDs:

- Log rotation policy and retention windows.
- Mandatory key schema pinning for JSON records.
- Optional additional sinks (for example mirrored stderr).

## Acceptance Scenarios

### Scenario SCN-0000: Writes application logs to a derived sigil.log file path

Given an effective `log_dir` value  
When application logging is initialized  
Then the effective log file path is `<log_dir>/sigil.log`.

### Scenario SCN-0001: Uses JSON structured log records for application logging

Given application logging emits records  
When log entries are written  
Then records are emitted in structured JSON format.

### Scenario SCN-0002: Uses default log file path when default log_dir is in effect

Given default `PRD-0001` application configuration values  
When logging is initialized  
Then the effective log target path is `./sigil/logs/sigil.log`.

### Scenario SCN-0003: Fails initialization when derived log file path cannot be opened as a file sink

Given a derived log file path that cannot be opened/created as a file sink  
When logging initialization runs  
Then startup/initialization fails with non-zero exit.

### Scenario SCN-0004: Derives log file path from configured log_dir override

Given an overridden `log_dir` value  
When logging is initialized  
Then the effective log target path is `<overridden-log_dir>/sigil.log`.
