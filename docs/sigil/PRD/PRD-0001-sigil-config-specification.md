# PRD-0001: Sigil Application Configuration

## Status

Accepted

## Context

`sigil` requires an application-level configuration baseline that is separate
from run configuration. This PRD defines the baseline application configuration
schema, defaults, environment override behavior, and validation contract.

## Goals

- Define the default application configuration file path and format.
- Define baseline application configuration keys and defaults.
- Define baseline environment override behavior for baseline keys.
- Define baseline validation behavior for application configuration.

## Non-Goals

- Defining the run configuration schema.
- Defining advanced application configuration keys beyond the baseline.
- Defining run lifecycle behavior or run payload structure.

## Application Config Source Contract

- The default application configuration file MUST be `./sigil.yaml` (current
  working directory).
- The application configuration format MUST be YAML.

## Baseline Application Schema

The baseline application configuration schema is:

```yaml
log_level: <string>
log_dir: <string>
```

## Default Values

- `log_level`: `info`
- `log_dir`: `./sigil/logs`

## Environment Namespace Contract

- The application configuration environment namespace MUST use `SIGIL`.
- Dot-to-underscore key mapping
  (`strings.NewReplacer(".", "_")`) MUST be used for application
  configuration keys.
- Baseline environment override keys are:
  - `SIGIL_LOG_LEVEL`
  - `SIGIL_LOG_DIR`
- Merge precedence for baseline keys MUST be:
  1. Environment overrides
  2. File values from `./sigil.yaml`
  3. Defaults

## Validation Rules

- `log_level` MUST be one of: `debug`, `info`, `warn`, `error`.
- Unsupported `log_level` values MUST fail configuration initialization with a
  validation error.
- `log_level` validation is case-sensitive in this baseline contract.

## Path Resolution Rules

- `log_dir` is a string path.
- Relative `log_dir` values MUST be resolved from the current working directory.

## Deferred Contracts

The following are explicitly deferred to future PRDs:

- Additional application configuration keys beyond baseline.
- Advanced path validation (for example existence, permission, creation policy).
- Extended normalization/compatibility rules for `log_level` values.

## Acceptance Scenarios

### Scenario SCN-0000: Defaults application configuration to sigil.yaml and defines baseline app-config schema

Given the `sigil` application starts without an explicit application config path  
When application configuration is resolved  
Then the default file is `./sigil.yaml`, format is YAML, and baseline keys are
`log_level` and `log_dir`.

### Scenario SCN-0001: Applies defaults and SIGIL environment overrides for log_level and log_dir

Given missing or partial baseline key values in `sigil.yaml` and optional
`SIGIL_LOG_LEVEL`/`SIGIL_LOG_DIR` environment values  
When application configuration is merged  
Then defaults are `log_level=info` and `log_dir=./sigil/logs`, and environment
values override file/default values.

### Scenario SCN-0002: Rejects unsupported log_level values

Given merged application configuration with `log_level` outside
`debug|info|warn|error`  
When validation runs  
Then configuration initialization fails with a validation error.
