# ADR-0001: Use Cobra for Sigil CLI

## Status

Accepted

## Date

2026-03-01

## Context

`sigil` requires a command-line interface that supports nested command trees,
consistent flag handling, clear help output, and predictable validation flow as
the runtime and control-plane surface area grows.

The implementation needs a framework that supports:

- Structured command composition across multiple files and directories.
- Input validation before command execution.
- Standardized command documentation fields (`Use`, `Short`, `Long`,
  `Example`).
- Consistent behavior for usage output and runtime errors.

## Decision

`sigil` CLI MUST use `github.com/spf13/cobra` as its command framework.

## Decision Details

- The root command MUST be defined in `cmd/root.go`.
- Top-level parent commands MUST each have a dedicated directory:
  `cmd/<parent-command>/`.
- Parent-command subcommands MUST be placed under
  `cmd/<parent-command>/subcommands/`.
- Each subcommand MUST be defined in its own file in the `subcommands` package.
- Each `subcommands/` directory MUST include `helpers.go` for shared helpers.
- Every command except root MUST define `PreRunE` via a named validation
  function.
- `PreRunE` MUST validate arguments and flags before runtime logic executes.
- `cmd.SilenceUsage` MUST be set to `true` only after validation succeeds.
- Flags MUST be bound to package-scope variables using variable-based binding
  (`StringVar`, `BoolVar`, `IntVar`, `StringSliceVar`, etc.).
- Command implementations MUST use bound variables directly instead of repeated
  `cmd.Flags().Get*()` retrieval as the normal access pattern.
- Wrapped Cobra runtime errors SHOULD use semicolon punctuation with `%w`
  (for example `fmt.Errorf("failed to initialize config; %w", err)`), matching
  the CLI's `Error:` prefixed output style.

## Alternatives Considered

- Go standard library `flag`: rejected because it does not provide comparable
  hierarchical command ergonomics for the expected CLI shape.
- `urfave/cli`: rejected to avoid divergence from the selected team standard
  and documented command-structure conventions.
- Custom in-house command router: rejected due to avoidable maintenance and
  documentation overhead.

## Consequences

- `sigil` command structure, validation flow, and help/usage conventions become
  standardized around Cobra.
- New commands must follow the documented directory and validation patterns.
- Deviating from Cobra in future requires a superseding ADR with migration and
  compatibility guidance.

## Migration/Adoption Notes

- New CLI surface area should be implemented directly on Cobra.
- Any pre-existing ad-hoc command scaffolding should be migrated to Cobra
  incrementally when touched.
- This ADR establishes architecture direction only; it does not itself mutate
  runtime implementation files.

## Related Documents

- [Sigil ADR Index](README.md)
- [Sigil Subproject README](../../../../sigil/README.md)
- [Project Workflow Rules](../../../../.agents/rules/WORKFLOW.md)
- [Project Spec Rules](../../../../.agents/rules/SPECS.md)
