# PRD-0120: Sigil Run Configuration Core Specification

## Status

Accepted

## Context

`sigil` needs a deterministic run-configuration contract that defines how core
run settings are loaded, merged, and validated before execution begins.

This PRD owns the run-configuration core:

- source resolution
- merge precedence
- prompt and context selection
- gateway, provider, and model selection
- recursive harness baseline settings

Cross-cutting extensions are defined in dedicated PRDs:

- reasoning request behavior: `PRD-0300`
- prompt composition and schema rendering: `PRD-0320`
- deterministic guardrails: `PRD-0500`
- accounting pricing and rollups: `PRD-0510`

## Goals

- Define the external run configuration file source and merge precedence.
- Define the core run configuration schema for prompt, context, LLM, and RLM settings.
- Define default values for the core run configuration surface.
- Define validation rules for mutually exclusive fields and supported provider or model pairs.

## Non-Goals

- Defining guardrail config semantics beyond section ownership and integration.
- Defining accounting pricing semantics beyond section ownership and integration.
- Defining inference transport behavior beyond selecting the configured gateway.
- Defining runtime implementation details outside configuration loading and validation.

## Run Configuration Source and Resolution

- The default external run configuration file MUST be `./sigil-run.yaml`.
- The run configuration format MUST be YAML.
- Environment overrides MUST be applied at runtime via Viper automatic environment support.
- Run configuration merge precedence MUST be:
  1. environment overrides
  2. file values from `./sigil-run.yaml`
  3. defaults
- Missing `./sigil-run.yaml` is allowed as long as the merged run configuration still satisfies all required fields and validation rules.

## Core Run Configuration Schema

The run configuration root is:

```yaml
system_prompt_append: <string>
prompt: <string>
prompt_template: <string>
context: <string>
context_template: <string>
llm:
  provider: <string>
  model: <string>
  gateway: <string>
  reasoning: <section defined by PRD-0300>
  openrouter:
    base_url: <string>
    request_timeout_ms: <int>
    api_key_env: <string>
rlm:
  enabled: <bool>
  max_depth: <int>
guardrails: <section defined by PRD-0500>
accounting: <section defined by PRD-0510>
```

Ownership rules:

- This PRD owns `system_prompt_append`, `prompt`, `prompt_template`, `context`, `context_template`, `llm.provider`, `llm.model`, `llm.gateway`, `llm.openrouter.*`, and `rlm.*`.
- `llm.reasoning` remains part of the run config shape, but its behavioral semantics are defined in `PRD-0300`.
- `guardrails` section semantics are defined in `PRD-0500`.
- `accounting` section semantics are defined in `PRD-0510`.

## Default Values

- `llm.provider`: no default, required
- `llm.model`: no default, required
- `system_prompt_append`: empty string
- `prompt`: no default
- `prompt_template`: no default
- `context`: no default
- `context_template`: no default
- `llm.gateway`: `openrouter`
- `llm.openrouter.base_url`: `https://openrouter.ai/api/v1`
- `llm.openrouter.request_timeout_ms`: `30000`
- `llm.openrouter.api_key_env`: `OPENROUTER_API_KEY`
- `rlm.enabled`: `true`
- `rlm.max_depth`: `3`

Defaults for reasoning, guardrails, and accounting are defined in their owning PRDs.

## Core Validation Rules

- Exactly one of `prompt` and `prompt_template` MUST be set.
- Exactly one of `context` and `context_template` MUST be set.
- `llm.provider` MUST be set after merge.
- `llm.model` MUST be set after merge.
- `llm.gateway` MUST be `openrouter` for this release.
- `llm.openrouter` MAY be omitted when `llm.gateway` is `openrouter`; effective fields MUST resolve from merge precedence.
- `llm.provider` MUST be one of:
  - `openai`
  - `anthropic`
- If `llm.provider=openai`, `llm.model` MUST be one of:
  - `gpt-5.1`
  - `gpt-5.1-codex-max`
  - `gpt-5.2`
  - `gpt-5.2-pro`
  - `gpt-5.2-codex`
  - `gpt-5.3-codex`
- If `llm.provider=anthropic`, `llm.model` MUST be one of:
  - `claude-sonnet-4`
  - `claude-opus-4`
  - `claude-sonnet-4.5`
  - `claude-haiku-4.5`
  - `claude-opus-4.5`
  - `claude-sonnet-4.6`
  - `claude-opus-4.6`

## Environment Override Contract

Viper runtime environment configuration MUST be initialized with:

- `SetEnvPrefix("SIGIL_RUN")`
- `SetEnvKeyReplacer(strings.NewReplacer(".", "_"))`
- `AutomaticEnv()`

Representative environment variables include:

- `SIGIL_RUN_SYSTEM_PROMPT_APPEND`
- `SIGIL_RUN_PROMPT`
- `SIGIL_RUN_PROMPT_TEMPLATE`
- `SIGIL_RUN_CONTEXT`
- `SIGIL_RUN_CONTEXT_TEMPLATE`
- `SIGIL_RUN_LLM_PROVIDER`
- `SIGIL_RUN_LLM_MODEL`
- `SIGIL_RUN_LLM_GATEWAY`
- `SIGIL_RUN_LLM_OPENROUTER_BASE_URL`
- `SIGIL_RUN_LLM_OPENROUTER_REQUEST_TIMEOUT_MS`
- `SIGIL_RUN_LLM_OPENROUTER_API_KEY_ENV`
- `SIGIL_RUN_RLM_ENABLED`
- `SIGIL_RUN_RLM_MAX_DEPTH`

## Implementation Constraints

- Configuration types MUST be defined in `internal/config/types.go`.
- Config fields MUST use `yaml` struct tags.
- Nested YAML keys MUST be represented using nested structs in `types.go`.
- Default values MUST be defined in `internal/config/defaults.go`.
- Bootstrap and merge orchestration MUST use Viper and enforce the precedence and validation rules defined in this PRD.

## Acceptance Scenarios

### Scenario SCN-0000: Loads default run configuration file from current working directory

Given no explicit run-config path override  
When `sigil` initializes run configuration  
Then it reads run configuration from `./sigil-run.yaml` by default.

### Scenario SCN-0001: Applies SIGIL_RUN environment variable overrides using dot-to-underscore key mapping

Given run configuration values in `sigil-run.yaml` and corresponding
`SIGIL_RUN_*` environment variables  
When run configuration is merged  
Then environment values override file values using dot-to-underscore key mapping.

### Scenario SCN-0002: Rejects run configuration when llm.provider or llm.model is missing after merge

Given merged run configuration omits `llm.provider` or `llm.model`  
When validation runs  
Then run configuration is rejected.

### Scenario SCN-0003: Requires exactly one of prompt and prompt_template

Given run configuration provides both or neither of `prompt` and
`prompt_template`  
When validation runs  
Then run configuration is rejected.

### Scenario SCN-0004: Requires exactly one of context and context_template

Given run configuration provides both or neither of `context` and
`context_template`  
When validation runs  
Then run configuration is rejected.

### Scenario SCN-0005: Defaults llm.gateway to openrouter when omitted

Given run configuration omits `llm.gateway`  
When defaults are applied  
Then `llm.gateway` resolves to `openrouter`.

### Scenario SCN-0006: Allows omitted llm.openrouter block when gateway is openrouter

Given run configuration sets `llm.gateway=openrouter` and omits `llm.openrouter`  
When configuration is merged  
Then effective OpenRouter fields resolve from defaults and environment overrides.

### Scenario SCN-0007: Rejects unsupported llm.gateway values

Given merged run configuration sets `llm.gateway` to a value other than
`openrouter`  
When validation runs  
Then run configuration is rejected.

### Scenario SCN-0008: Applies OpenRouter defaults when openrouter fields are omitted

Given run configuration omits one or more `llm.openrouter` fields  
When defaults are applied  
Then omitted OpenRouter fields resolve to their default values.

### Scenario SCN-0009: Applies RLM defaults when rlm fields are omitted

Given run configuration omits one or more `rlm` fields  
When defaults are applied  
Then omitted RLM fields resolve to their default values.

### Scenario SCN-0010: Accepts required values provided entirely by environment variables

Given no `sigil-run.yaml` file is present  
And required run configuration values are provided entirely by `SIGIL_RUN_*`
environment variables  
When configuration is initialized  
Then merged run configuration is accepted.

### Scenario SCN-0011: Accepts llm.provider only when value is openai or anthropic

Given merged run configuration sets `llm.provider`  
When validation runs  
Then only `openai` and `anthropic` are accepted.

### Scenario SCN-0012: Validates llm.model against allowed list for provider openai

Given merged run configuration sets `llm.provider=openai`  
When validation runs  
Then `llm.model` must be in the allowed OpenAI model list.

### Scenario SCN-0013: Validates llm.model against allowed list for provider anthropic

Given merged run configuration sets `llm.provider=anthropic`  
When validation runs  
Then `llm.model` must be in the allowed Anthropic model list.

### Scenario SCN-0014: Rejects run configuration when llm.model is not allowed for selected llm.provider

Given merged run configuration selects a provider and an unsupported model for
that provider  
When validation runs  
Then run configuration is rejected.
