# PRD-0110: Sigil Initial CLI Surface

## Status

Accepted

## Context

`sigil` requires a stable initial command-line surface so users and downstream
integrations can discover command structure while run lifecycle behavior evolves.

This PRD defines the external CLI command tree contract and usage behavior for
the root and parent command surface.

## Goals

- Define the initial `sigil` executable command tree.
- Define observable behavior for root and parent commands and delegated command
  entrypoints.
- Define initial exit code behavior for usage-only command paths.
- Define error handling expectations for unknown or invalid subcommands.

## Non-Goals

- Defining run lifecycle execution behavior internals for `run start` or
  `run stop`.
- Defining `run stop` positional-argument or execution semantics beyond
  delegation.
- Defining custom error strings beyond Cobra default behavior.

## CLI Surface Contract

The initial CLI surface MUST be:

```text
sigil
└── run
    ├── start
    └── stop
```

- Executable name MUST be `sigil`.
- `run` MUST be a subcommand of the root command.
- `start` and `stop` MUST be subcommands of `run`.

## Command Behavior Contract

- `sigil` MUST print usage/help and perform no runtime action.
- `sigil run` MUST print usage/help and perform no runtime action.
- `sigil run start` behavior is delegated to
  `PRD-0410-sigil-run-start-command-execution-specification.md`.
- `sigil run stop` behavior is delegated to
  `PRD-0150-sigil-run-stop-command-inputs-specification.md` and
  `PRD-0450-sigil-run-stop-command-execution-specification.md`.
- No positional arguments or required flags are defined in this PRD.

## Exit Code Contract

- `sigil` with no subcommand MUST exit with status code `0`.
- `sigil run` with no subcommand MUST exit with status code `0`.
- `sigil run start` exit behavior is defined by
  `PRD-0410-sigil-run-start-command-execution-specification.md`.
- `sigil run stop` exit behavior is defined by
  `PRD-0450-sigil-run-stop-command-execution-specification.md`.

## Error Handling Contract

- Unknown subcommands and invalid usage paths MUST use Cobra default behavior.
- This PRD does not require exact error text for framework-default paths.
- Unknown or invalid subcommand paths SHOULD exit non-zero under framework
  defaults.

## Deferred Contracts

The following are explicitly deferred to a future PRD:

- Lifecycle, event, and interruption internals owned by runtime lifecycle and
  event PRDs.

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

### Scenario SCN-0003: Delegates sigil run start behavior to PRD-0410 run-start command-execution contract

Given the `sigil` executable is available  
When a user runs `sigil run start`  
Then `sigil run start` behavior follows
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
