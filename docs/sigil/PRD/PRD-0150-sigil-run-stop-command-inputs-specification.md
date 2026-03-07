# PRD-0150: Sigil Run Stop Command Inputs Specification

## Status

Draft

## Context

`sigil run stop` needs a stable command-input contract before its local control
transport and graceful-interruption runtime behavior can be relied on.

This PRD defines the required positional argument and validation behavior for
`sigil run stop`.

## Goals

- Define the canonical `sigil run stop <run-id>` invocation form.
- Define argument-count and identifier validation behavior.
- Define baseline command-validation failure behavior.

## Non-Goals

- Defining stop-request transport or signal behavior.
- Defining stop-command success output shape.
- Defining run-state or event-log semantics after validation succeeds.

## Command Input Contract

- `sigil run stop` MUST be invoked as `sigil run stop <run-id>`.
- `run-id` MUST be a positional argument, not a flag.
- `sigil run stop` MUST require exactly one positional argument.
- The provided `run-id` MUST validate as UUIDv7 before any filesystem or
  process lookup occurs.
- No `--run-id` flag is introduced in v1.
- No stop-specific flags are introduced in v1.

## Validation and Error Contract

- Missing `run-id` input MUST fail command validation with non-zero exit.
- Extra positional arguments MUST fail command validation with non-zero exit.
- Non-UUIDv7 `run-id` values MUST fail command validation with non-zero exit.
- Unknown or unsupported flags SHOULD follow Cobra default behavior and SHOULD
  exit non-zero.

## Deferred Contracts

The following are explicitly deferred to future PRDs:

- Stop-command success output shape.
- Stop-request transport and wait behavior.
- Run discovery or listing behavior.

## Acceptance Scenarios

### Scenario SCN-0000: Requires exactly one UUIDv7 run-id positional argument for sigil run stop

Given `sigil run stop` command inputs are evaluated  
When the command is invoked with missing extra or non-UUIDv7 `run-id` input  
Then command validation fails with non-zero exit.
