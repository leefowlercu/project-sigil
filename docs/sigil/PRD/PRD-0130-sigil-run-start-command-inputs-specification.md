# PRD-0130: Sigil Run Start Command Inputs Specification

## Status

Accepted

## Context

`sigil run start` needs a stable input contract for selecting configuration
files, template variables, and run storage location before runtime execution
begins.

This PRD defines baseline run-start flags, path resolution, and validation
behavior for command initialization.

## Goals

- Define baseline `sigil run start` config-path flags.
- Define inherited run-storage selection behavior for `sigil run start`.
- Define precedence and coexistence behavior when multiple inputs are passed.
- Define baseline validation and failure behavior before run execution begins.

## Non-Goals

- Defining run execution semantics after configuration is accepted.
- Defining run-start success payload or run identifier output.
- Defining YAML parse/schema validation behavior for config file contents.

## Command Input Contract

- `sigil run start` MUST accept `--config <path>` as an optional flag targeting
  a `sigil` application config file source.
- `sigil run start` MUST accept `--run-config <path>` as an optional flag
  targeting a `sigil` run config file source.
- `sigil run start` MUST accept inherited `-o, --output <text|json>` values
  from the root CLI surface defined in `PRD-0110`.
- `sigil run start` MUST accept inherited `--run-dir <path>` values from the
  `sigil run` parent command.
- `sigil run start` MUST accept repeatable `--var <key=value>` flags for
  template variable injection used by runtime behavior in
  `PRD-0410-sigil-run-start-command-execution-specification.md`.
- Both config-path flags MAY be supplied together.

## Path Resolution Contract

- When `--config` is omitted, the resolved application config path MUST default
  to `./sigil.yaml`.
- When `--run-config` is omitted, the resolved run config path MUST default to
  `./sigil-run.yaml`.
- When inherited `--run-dir` is omitted, the resolved run storage base
  directory MUST default to `./.sigil/runs`.
- When `--config` is provided, it MUST override only the application config
  path.
- When `--run-config` is provided, it MUST override only the run config path.
- When inherited `--run-dir` is provided, it MUST override only the run
  storage base directory.
- Merge and selection precedence for path inputs MUST be: explicit flag value,
  then command default.

## Validation and Error Contract

- The resolved application configuration path MUST exist.
- The resolved application configuration path MUST be a readable regular file
  (not a directory).
- The resolved run configuration path MUST be a readable regular file when
  `--run-config` is explicitly provided.
- When `--run-config` is omitted, a missing default `./sigil-run.yaml` MUST be
  allowed as long as run configuration merge and validation rules still pass.
- An explicitly provided inherited `--run-dir` value MUST NOT be empty or
  whitespace-only.
- A `--var` entry MUST include one non-empty key and one value separated by the
  first `=` character.
- Invalid `--var` entries MUST fail command validation with non-zero exit.
- Duplicate `--var` keys MUST resolve deterministically using the last provided
  value.
- Invalid inherited `--output` values MUST fail command validation before
  configuration initialization or run execution begins.
- Unknown or unsupported flags SHOULD follow Cobra default behavior and SHOULD
  exit non-zero.

## Deferred Contracts

The following are explicitly deferred to future PRDs:

- Run-start success output shape.
- Run identifier emission contract.
- Run engine behavior after configuration initialization succeeds.
- Detailed parse/schema validation error contract for config file content.

## Acceptance Scenarios

### Scenario SCN-0000: Uses default application and run configuration paths when no flags are provided

Given `sigil run start` is invoked without `--config` and without
`--run-config`  
When configuration paths are resolved  
Then the application config path is `./sigil.yaml` and the run config path is
`./sigil-run.yaml`.

### Scenario SCN-0001: Overrides application configuration path with --config

Given `sigil run start --config <path>` is invoked without `--run-config`  
When configuration paths are resolved  
Then the application config path uses `<path>` and the run config path remains
`./sigil-run.yaml`.

### Scenario SCN-0002: Overrides run configuration path with --run-config

Given `sigil run start --run-config <path>` is invoked without `--config`  
When configuration paths are resolved  
Then the run config path uses `<path>` and the application config path remains
`./sigil.yaml`.

### Scenario SCN-0003: Overrides both configuration paths when both flags are provided

Given `sigil run start --config <app-path> --run-config <run-path>`  
When configuration paths are resolved  
Then the application config path uses `<app-path>` and the run config path uses
`<run-path>`.

### Scenario SCN-0004: Fails when required configuration paths are missing unreadable or not regular files

Given a resolved application configuration path is missing unreadable or not a
regular file  
Or an explicitly provided run configuration path is missing unreadable or not a
regular file  
When `sigil run start` initializes command inputs  
Then command initialization fails with non-zero exit  
And unknown or unsupported flags use framework-default Cobra error behavior.

### Scenario SCN-0005: Accepts repeated --var key value entries

Given `sigil run start` receives repeated `--var <key=value>` flags  
When command input validation runs  
Then repeated variable inputs are accepted.

### Scenario SCN-0006: Rejects invalid --var format or empty key

Given `sigil run start` receives one or more invalid `--var` entries  
When command input validation runs  
Then command fails validation with non-zero exit.

### Scenario SCN-0007: Resolves duplicate --var keys deterministically using last value

Given `sigil run start` receives duplicate `--var` keys in order  
When template variable map is resolved  
Then the last provided value for each duplicate key is used.

### Scenario SCN-0008: Rejects invalid inherited output value for sigil run start

Given `sigil run start` receives an inherited `--output` value outside
`text|json`  
When command input validation runs  
Then command validation fails with non-zero exit before run execution begins.

### Scenario SCN-0009: Overrides default run storage base directory with inherited --run-dir for sigil run start

Given `sigil run start` receives inherited `--run-dir <path>`  
When command inputs are resolved  
Then the effective run storage base directory uses `<path>`.

### Scenario SCN-0010: Rejects explicit empty inherited --run-dir value for sigil run start

Given `sigil run start` receives inherited `--run-dir` with an empty or
whitespace-only value  
When command input validation runs  
Then command validation fails with non-zero exit before run execution begins.
