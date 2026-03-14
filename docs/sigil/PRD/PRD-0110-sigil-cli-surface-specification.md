# PRD-0110: Sigil CLI Surface Specification

## Status

Accepted

## Context

`sigil` requires a stable command-line surface so users and downstream
integrations can discover execution, control, and inspection entrypoints
without inferring command structure from implementation details.

This PRD defines the external CLI command tree, inherited flags, and delegated
behavior ownership for the root, run, and app-server parent command surfaces.

## Goals

- Define the `sigil` executable command tree.
- Define inherited CLI flags available across the command tree.
- Define observable behavior for root and parent command entrypoints.
- Define delegation boundaries for run start, stop, inspection, and app-server
  commands.
- Define error handling expectations for unknown or invalid subcommands.

## Non-Goals

- Defining run lifecycle execution internals for `run start`.
- Defining stop transport internals for `run stop`.
- Defining read-model field semantics beyond delegated inspection PRDs.
- Defining custom error strings beyond Cobra default behavior.

## CLI Surface Contract

The CLI surface MUST be:

```text
sigil
├── run [--run-dir <path>]
│   ├── start
│   ├── stop
│   ├── list
│   ├── status <run-id>
│   ├── inspect <run-id>
│   └── events <run-id>
└── app-server
    ├── serve
    ├── generate-ts
    └── generate-json-schema
```

- Executable name MUST be `sigil`.
- `run` MUST be a subcommand of the root command.
- `app-server` MUST be a subcommand of the root command.
- `start`, `stop`, `list`, `status`, `inspect`, and `events` MUST be
  subcommands of `run`.
- `serve`, `generate-ts`, and `generate-json-schema` MUST be subcommands of
  `app-server`.

## Inherited Flag Contract

- `sigil` MUST accept a persistent `-o, --output <text|json>` flag.
- The root output flag MUST be inherited by `sigil run` and all run
  subcommands.
- The default output format MUST be `text`.
- `json` output mode is compatibility mode for command result payloads only.
- Help, usage, and Cobra-generated error surfaces MUST remain human-readable
  text even when `--output json` is present.
- `sigil run` MUST accept a persistent `--run-dir <path>` flag.
- The `run` parent `--run-dir` flag MUST be inherited by all run subcommands.
- When `--run-dir` is omitted, run commands MUST fall back to the default run
  storage directory owned by the runtime layer.
- `--run-dir` is scoped to the `run` command tree and MUST NOT be introduced as
  a root-global flag in v1.

## Command Behavior Contract

- `sigil` MUST print usage/help and perform no runtime action.
- `sigil run` MUST print usage/help and perform no runtime action.
- `sigil run start` behavior is delegated to
  `PRD-0130-sigil-run-start-command-inputs-specification.md` and
  `PRD-0410-sigil-run-start-command-execution-specification.md`.
- `sigil run stop` behavior is delegated to
  `PRD-0150-sigil-run-stop-command-inputs-specification.md` and
  `PRD-0450-sigil-run-stop-command-execution-specification.md`.
- `sigil run list`, `sigil run status`, `sigil run inspect`, and
  `sigil run events` behavior is delegated to
  `PRD-0160-sigil-run-inspection-command-specification.md` and
  `PRD-0220-sigil-run-projection-and-query-specification.md`.
- `sigil app-server serve`, `sigil app-server generate-ts`, and
  `sigil app-server generate-json-schema` behavior is delegated to
  `PRD-0170-sigil-app-server-transport-and-handshake-specification.md`.
- No positional arguments or required flags are defined for `sigil` or
  `sigil run` in this PRD.

## Exit Code Contract

- `sigil` with no subcommand MUST exit with status code `0`.
- `sigil run` with no subcommand MUST exit with status code `0`.
- `sigil run start` exit behavior is defined by
  `PRD-0410-sigil-run-start-command-execution-specification.md`.
- `sigil run stop` exit behavior is defined by
  `PRD-0450-sigil-run-stop-command-execution-specification.md`.
- `sigil run list`, `sigil run status`, `sigil run inspect`, and
  `sigil run events` exit behavior is defined by
  `PRD-0160-sigil-run-inspection-command-specification.md`.
- `sigil app-server serve`, `sigil app-server generate-ts`, and
  `sigil app-server generate-json-schema` exit behavior is defined by
  `PRD-0170-sigil-app-server-transport-and-handshake-specification.md`.

## Error Handling Contract

- Unknown subcommands and invalid usage paths MUST use Cobra default behavior.
- This PRD does not require exact error text for framework-default paths.
- Unknown or invalid subcommand paths SHOULD exit non-zero under framework
  defaults.

## Deferred Contracts

The following are explicitly deferred to future PRDs:

- Lifecycle, event, and interruption internals owned by runtime lifecycle and
  event PRDs.
- Remote transports or method families beyond the initial `app-server`
  subcommands.

## Acceptance Scenarios

### Scenario SCN-0000: Exposes the sigil executable entrypoint

Given the `sigil` application is installed  
When a user invokes `sigil --help`  
Then the CLI is available via the executable name `sigil`.

### Scenario SCN-0001: Prints root usage when sigil is invoked without subcommands

Given the `sigil` executable is available  
When a user runs `sigil`  
Then root usage/help is printed and the process exits with status code `0`.

### Scenario SCN-0002: Prints run usage when sigil run is invoked without subcommands

Given the `sigil` executable is available  
When a user runs `sigil run`  
Then run-subcommand usage/help is printed and the process exits with status
code `0`.

### Scenario SCN-0003: Delegates sigil run start behavior to PRD-0130 and PRD-0410 run-start contracts

Given the `sigil` executable is available  
When a user runs `sigil run start`  
Then `sigil run start` behavior follows
`PRD-0130-sigil-run-start-command-inputs-specification.md` and
`PRD-0410-sigil-run-start-command-execution-specification.md`.

### Scenario SCN-0004: Delegates sigil run stop behavior to PRD-0150 and PRD-0450 run-stop contracts

Given the `sigil` executable is available  
When a user runs `sigil run stop`  
Then `sigil run stop` behavior follows
`PRD-0150-sigil-run-stop-command-inputs-specification.md` and
`PRD-0450-sigil-run-stop-command-execution-specification.md`.

### Scenario SCN-0005: Uses framework-default error behavior for unknown subcommands

Given the `sigil` executable is available  
When a user runs a command path with an unknown subcommand  
Then the CLI uses Cobra default error and usage behavior and exits non-zero.

### Scenario SCN-0006: Keeps root and run help text when output json is requested

Given the `sigil` executable is available  
When a user runs `sigil --output json` or `sigil run --output json`  
Then root and run help surfaces remain human-readable text  
And the process exits with status code `0`.

### Scenario SCN-0007: Exposes run inspection subcommands under sigil run

Given the `sigil` executable is available  
When a user inspects `sigil run --help`  
Then the command surface exposes `list`, `status`, `inspect`, and `events`
subcommands.

### Scenario SCN-0008: Inherits run storage override flag across sigil run subcommands

Given the `sigil` executable is available  
When a user inspects `sigil run --help` or one of its subcommands  
Then the run-subcommand surface documents inherited `--run-dir <path>`
behavior.
