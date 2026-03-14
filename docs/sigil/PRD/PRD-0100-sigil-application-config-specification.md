# PRD-0100: Sigil Application Configuration Specification

## Status

Accepted

## Context

`sigil` requires an application-level configuration contract that stays separate
from run configuration. The initial baseline covered only flat logging keys.

This PRD now defines the grouped application configuration baseline used by both
the CLI bootstrap path and the emerging app-server process surface.

## Goals

- Define the default application configuration file path and format.
- Define grouped `logs` and `app_server` configuration sections.
- Define environment override behavior for grouped application keys.
- Define baseline validation and path-resolution behavior for application
  configuration.

## Non-Goals

- Defining the run configuration schema.
- Defining remote auth, TLS, or non-localhost deployment policy.
- Defining app-server execution and subscription behavior beyond configuration.

## Application Config Source Contract

- The default application configuration file MUST be `./sigil.yaml` from the
  current working directory.
- The application configuration format MUST be YAML.

## Baseline Application Schema

The grouped application configuration baseline is:

```yaml
logs:
  level: <string>
  dir: <string>

app_server:
  instance_name: <string>
  instance_id: <string>
  run_dir: <string>
  allowed_origins:
    - <origin>
  websocket:
    listen_addr: <host:port>
    path: <string>
  health:
    ready_path: <string>
    live_path: <string>
  subscriptions:
    poll_interval_ms: <int>
  limits:
    max_connections: <int>
    max_frame_bytes: <int>
```

## Default Values

- `logs.level`: `info`
- `logs.dir`: `./.sigil/logs`
- `app_server.instance_name`: `sigil-local`
- `app_server.instance_id`: `sigil-local`
- `app_server.run_dir`: `./.sigil/runs`
- `app_server.allowed_origins`: empty list
- `app_server.websocket.listen_addr`: `127.0.0.1:8765`
- `app_server.websocket.path`: `/app-server`
- `app_server.health.ready_path`: `/readyz`
- `app_server.health.live_path`: `/healthz`
- `app_server.subscriptions.poll_interval_ms`: `500`
- `app_server.limits.max_connections`: `50`
- `app_server.limits.max_frame_bytes`: `1048576`

## Environment Namespace Contract

- The application configuration environment namespace MUST use `SIGIL`.
- Dot-to-underscore key mapping
  (`strings.NewReplacer(".", "_")`) MUST be used for application
  configuration keys.
- Baseline environment override keys include:
  - `SIGIL_LOGS_LEVEL`
  - `SIGIL_LOGS_DIR`
  - `SIGIL_APP_SERVER_INSTANCE_NAME`
  - `SIGIL_APP_SERVER_INSTANCE_ID`
  - `SIGIL_APP_SERVER_RUN_DIR`
  - `SIGIL_APP_SERVER_WEBSOCKET_LISTEN_ADDR`
  - `SIGIL_APP_SERVER_WEBSOCKET_PATH`
  - `SIGIL_APP_SERVER_HEALTH_READY_PATH`
  - `SIGIL_APP_SERVER_HEALTH_LIVE_PATH`
  - `SIGIL_APP_SERVER_SUBSCRIPTIONS_POLL_INTERVAL_MS`
  - `SIGIL_APP_SERVER_LIMITS_MAX_CONNECTIONS`
  - `SIGIL_APP_SERVER_LIMITS_MAX_FRAME_BYTES`
- Merge precedence for application keys MUST be:
  1. Environment overrides
  2. File values from `./sigil.yaml`
  3. Built-in defaults

## Validation Rules

- `logs.level` MUST be one of: `debug`, `info`, `warn`, `error`.
- Unsupported `logs.level` values MUST fail configuration initialization with a
  validation error.
- `app_server.instance_name` and `app_server.instance_id` MUST be non-empty.
- `app_server.websocket.path`, `app_server.health.ready_path`, and
  `app_server.health.live_path` MUST start with `/`.
- `app_server.subscriptions.poll_interval_ms`,
  `app_server.limits.max_connections`, and
  `app_server.limits.max_frame_bytes` MUST be positive integers.
- `app_server.allowed_origins` entries, when present, MUST include scheme and
  host.

## Path Resolution Rules

- Relative `logs.dir` values MUST be resolved from the current working
  directory.
- Relative `app_server.run_dir` values MUST be resolved from the current
  working directory.

## Deferred Contracts

The following are explicitly deferred to future PRDs:

- Compatibility aliases for the retired flat `log_level` and `log_dir` keys.
- Multi-root app-server run discovery.
- Advanced origin, listener, or deployment validation beyond the grouped
  baseline above.

## Acceptance Scenarios

### Scenario SCN-0000: Defaults application configuration to sigil.yaml and defines baseline app-config schema

Given the `sigil` application starts without an explicit application config path  
When application configuration is resolved  
Then the default file is `./sigil.yaml`, format is YAML, and baseline keys are
`logs.level` and `logs.dir`.

### Scenario SCN-0001: Applies defaults and SIGIL environment overrides for logs.level and logs.dir

Given missing or partial baseline logging values in `sigil.yaml` and optional
`SIGIL_LOGS_LEVEL`/`SIGIL_LOGS_DIR` environment values  
When application configuration is merged  
Then defaults are `logs.level=info` and `logs.dir=./.sigil/logs`, and
environment values override file/default values.

### Scenario SCN-0002: Rejects unsupported logs.level values

Given merged application configuration with `logs.level` outside
`debug|info|warn|error`  
When validation runs  
Then configuration initialization fails with a validation error.
