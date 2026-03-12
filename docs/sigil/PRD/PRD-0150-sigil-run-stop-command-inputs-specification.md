# PRD-0150: Sigil Run Stop Command Inputs Specification

## Status

Draft

## Context

`sigil run stop` needs a stable command-input contract before its local control
transport and graceful-interruption runtime behavior can be relied on.

This PRD defines the required positional argument, inherited run-storage
selection, and validation behavior for `sigil run stop`.

## Goals

- Define the canonical `sigil run stop <run-id>` invocation form.
- Define inherited run-storage selection behavior for `sigil run stop`.
- Define argument-count and identifier validation behavior.
- Define baseline command-validation failure behavior.

## Non-Goals

- Defining stop-request transport or signal behavior.
- Defining stop-command success output shape.
- Defining run-state or event-log semantics after validation succeeds.

## Command Input Contract

- `sigil run stop` MUST be invoked as `sigil run stop <run-id>`.
- `sigil run stop` MUST accept inherited `-o, --output <text|json>` values from
  the root CLI surface defined in `PRD-0110`.
- `sigil run stop` MUST accept inherited `--run-dir <path>` values from the
  `sigil run` parent command.
- When inherited `--run-dir` is omitted, the effective run storage base
  directory MUST default to `./.sigil/runs`.
- `run-id` MUST be a positional argument, not a flag.
- `sigil run stop` MUST require exactly one positional argument.
- The provided `run-id` MUST validate as UUIDv7 before any filesystem or
  process lookup occurs.
- An explicitly provided inherited `--run-dir` value MUST NOT be empty or
  whitespace-only.
- No `--run-id` flag is introduced in v1.
- No stop-specific flags beyond inherited `--output` and `--run-dir` are
  introduced in v1.

## Validation and Error Contract

- Missing `run-id` input MUST fail command validation with non-zero exit.
- Extra positional arguments MUST fail command validation with non-zero exit.
- Non-UUIDv7 `run-id` values MUST fail command validation with non-zero exit.
- Invalid inherited `--output` values MUST fail command validation before any
  run-state lookup occurs.
- Explicit empty or whitespace-only inherited `--run-dir` values MUST fail
  command validation before any run-state lookup occurs.
- Unknown or unsupported flags SHOULD follow Cobra default behavior and SHOULD
  exit non-zero.

## Deferred Contracts

The following are explicitly deferred to future PRDs:

- Stop-command success output shape.
- Stop-request transport and wait behavior.
- Remote stop transport.

## Acceptance Scenarios

### Scenario SCN-0000: Requires exactly one UUIDv7 run-id positional argument for sigil run stop

Given `sigil run stop` command inputs are evaluated  
When the command is invoked with missing extra or non-UUIDv7 `run-id` input  
Then command validation fails with non-zero exit.

### Scenario SCN-0001: Rejects invalid inherited output value for sigil run stop

Given `sigil run stop` receives an inherited `--output` value outside
`text|json`  
When command input validation runs  
Then command validation fails with non-zero exit before any run lookup occurs.

### Scenario SCN-0002: Overrides default run storage base directory with inherited --run-dir for sigil run stop

Given `sigil run stop` receives inherited `--run-dir <path>`  
When command inputs are resolved  
Then the effective run storage base directory uses `<path>`.

### Scenario SCN-0003: Rejects explicit empty inherited --run-dir value for sigil run stop

Given `sigil run stop` receives inherited `--run-dir` with an empty or
whitespace-only value  
When command input validation runs  
Then command validation fails with non-zero exit before any run lookup occurs.
