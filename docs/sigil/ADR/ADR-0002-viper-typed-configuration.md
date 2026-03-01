# ADR-0002: Use Typed Viper Configuration

## Status

Accepted

## Date

2026-03-01

## Context

`sigil` requires configuration that is explicit, type-safe, environment-aware,
and maintainable as configuration domains grow. Direct string-key lookups can
hide schema drift and increase runtime configuration errors.

The implementation needs a configuration approach that supports:

- A typed schema as the primary access pattern.
- Defaults plus environment overrides.
- Centralized initialization and validation.
- Clear separation of config types, defaults, and bootstrap logic.

## Decision

`sigil` configuration MUST use `github.com/spf13/viper` with typed configuration
structs as the primary API.

## Decision Details

- `sigil` MUST define a dedicated config package under `internal/config/`.
- The config package MUST include:
  - `types.go`
  - `defaults.go`
  - `config.go`
- `types.go` MUST define the root `Config` struct and nested section structs,
  with `yaml` and `mapstructure` tags on configuration fields.
- `defaults.go` MUST define default constants and a `NewDefaultConfig()`
  constructor returning a fully populated `Config`.
- `config.go` MUST own configuration bootstrap and access APIs:
  `Init()`, `Get()`, `MustGet()`, and `ExpandPath()`.
- `config.go` MUST initialize Viper with config name/type, environment prefix,
  environment key replacer, and `viper.AutomaticEnv()`.
- Defaults MUST be registered before reading/unmarshaling configuration.
- Missing config files MUST NOT fail startup when defaults plus env overrides are
  sufficient.
- Active configuration MUST be unmarshaled into typed structs and validated
  before storage.
- Active configuration state MUST remain encapsulated within the config package.
- Environment variables MUST override file values and map cleanly to typed
  fields.
- String-key access (for example `viper.GetString("section.field")`) MUST NOT be
  the primary configuration access pattern.

## Alternatives Considered

- Raw `os.Getenv` plus custom parsing: rejected because it duplicates config
  merge/parsing concerns and weakens schema centralization.
- Map-based untyped config access: rejected because it increases runtime type
  errors and weakens IDE/refactor support.
- String-key-first Viper usage: rejected in favor of typed contracts.

## Consequences

- `sigil` configuration behavior is standardized around typed schemas and
  centralized initialization.
- Config changes become easier to review because schema and defaults are explicit
  in dedicated files.
- Deviations from typed Viper patterns require a superseding ADR with explicit
  rationale.

## Migration/Adoption Notes

- Configuration additions should be implemented by extending typed structs and
  defaults first, then wiring bootstrap behavior in `config.go`.
- Existing or transitional string-key access paths should be removed as touched.
- This ADR sets architecture direction only and does not itself apply runtime
  refactors.

## Related Documents

- [Sigil ADR Index](README.md)
- [Sigil Subproject README](../../../../sigil/README.md)
- [Project Spec Rules](../../../../.agents/rules/SPECS.md)
