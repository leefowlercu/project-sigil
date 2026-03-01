# PRD-0002: Sigil Initial CLI Surface

## Status

Accepted

## Context

`sigil` requires a stable initial command-line surface so users and downstream
integrations can discover command structure before run lifecycle behavior is
implemented.

This PRD defines the initial external CLI contract and usage behavior for the
first command tree.

## Goals

- Define the initial `sigil` executable command tree.
- Define observable behavior for root, parent, and placeholder commands.
- Define initial exit code behavior for usage-only command paths.
- Define error handling expectations for unknown or invalid subcommands.

## Non-Goals

- Defining run lifecycle execution behavior for `run start` or `run stop`.
- Defining `run stop --run-id` or any required flag contract.
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
- `sigil run start` behavior is superseded by
  `PRD-0004-sigil-run-start-cli-subcommand-specification.md` and MUST follow the
  run-start config-input contract defined there.
- `sigil run stop` MUST print usage/help as a usage-only placeholder command.
- No positional arguments or required flags are defined in this PRD.

## Exit Code Contract

- `sigil` with no subcommand MUST exit with status code `0`.
- `sigil run` with no subcommand MUST exit with status code `0`.
- `sigil run start` exit behavior is defined by
  `PRD-0004-sigil-run-start-cli-subcommand-specification.md`.
- `sigil run stop` usage-only behavior MUST exit with status code `0`.

## Error Handling Contract

- Unknown subcommands and invalid usage paths MUST use Cobra default behavior.
- This PRD does not require exact error text for framework-default paths.
- Unknown or invalid subcommand paths SHOULD exit non-zero under framework
  defaults.

## Deferred Contracts

The following are explicitly deferred to a future PRD:

- Runtime behavior for starting a run (`sigil run start`).
- Runtime behavior for stopping an in-progress run (`sigil run stop`).
- Any `run stop --run-id` flag schema, requiredness, and validation rules.

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

### Scenario SCN-0003: Defers sigil run start behavior to PRD-0004 run-start config-input contract

Given the `sigil` executable is available  
When a user runs `sigil run start`  
Then `sigil run start` behavior follows
`PRD-0004-sigil-run-start-cli-subcommand-specification.md`, replacing usage-only
placeholder semantics.

### Scenario SCN-0004: Provides sigil run stop as a usage-only placeholder command

Given the `sigil` executable is available  
When a user runs `sigil run stop`  
Then usage/help is printed and the process exits with status code `0`.

### Scenario SCN-0005: Uses framework-default error behavior for unknown subcommands

Given the `sigil` executable is available  
When a user runs a command path with an unknown subcommand  
Then the CLI uses Cobra default error and usage behavior and exits non-zero.
