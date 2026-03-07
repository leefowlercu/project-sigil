# PRD-0003: Sigil Run Configuration

## Status

Accepted

## Context

`sigil` needs a deterministic run configuration contract that supports:

- External YAML configuration by default.
- Runtime environment variable overrides.
- Typed configuration structures that map nested YAML keys to nested structs.
- Explicit validation for mutually exclusive and required key relationships.
- Deterministic inference provider/model/reasoning constraints for gateway-based
  inference.

This PRD defines the run-configuration behavioral contract for how
configuration is loaded, merged, and validated for run execution.

## Goals

- Define the initial external run configuration schema for `sigil`.
- Define run configuration default values and required fields.
- Define merge precedence between defaults, file values, and environment
  overrides for run configuration.
- Define validation rules for mutually exclusive fields, gateway constraints,
  provider/model allow-lists, reasoning configuration, and accounting fallback
  pricing.
- Provide acceptance scenarios that can map directly to submodule acceptance
  coverage.

## Non-Goals

- Defining additional LLM gateways beyond `openrouter`.
- Defining backward compatibility for root-level `provider` or `model` keys.
- Defining runtime implementation details outside the run-configuration
  contract.

## Run Configuration Source and Resolution

- The default external run configuration file MUST be `./sigil-run.yaml`
  (current working directory).
- The run configuration format MUST be YAML.
- Environment overrides MUST be applied at runtime via Viper automatic
  environment support.
- Run configuration merge precedence MUST be:
  1. Environment overrides
  2. File values from `./sigil-run.yaml`
  3. Defaults
- Missing `./sigil-run.yaml` is allowed as long as the merged run configuration
  still satisfies all required fields and validation rules.

## Run Configuration Schema

The initial run configuration schema is:

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
  reasoning:
    enabled: <bool>
    effort: <string>
  openrouter:
    base_url: <string>
    request_timeout_ms: <int>
    api_key_env: <string>
rlm:
  enabled: <bool>
  max_depth: <int>
guardrails:
  max_steps_per_node: <int>
  max_total_steps_per_run: <int>
  max_run_duration_ms: <int>
  max_consecutive_step_failures: <int>
accounting:
  pricing_version: <string>
  fallback_pricing:
    <provider>:
      <model>:
        input_microusd_per_million_tokens: <int>
        output_microusd_per_million_tokens: <int>
        reasoning_microusd_per_million_tokens: <int>
```

## Default Values

- `llm.provider`: no default, required
- `llm.model`: no default, required
- `system_prompt_append`: default empty string
- `prompt`: no default
- `prompt_template`: no default
- `context`: no default
- `context_template`: no default
- `llm.gateway`: `openrouter`
- `llm.reasoning.enabled`: `true`
- `llm.reasoning.effort`: `medium`
- `llm.openrouter.base_url`: `https://openrouter.ai/api/v1`
- `llm.openrouter.request_timeout_ms`: `30000`
- `llm.openrouter.api_key_env`: `OPENROUTER_API_KEY`
- `rlm.enabled`: `true`
- `rlm.max_depth`: `3`
- `guardrails.max_steps_per_node`: `64`
- `guardrails.max_total_steps_per_run`: `256`
- `guardrails.max_run_duration_ms`: `1200000`
- `guardrails.max_consecutive_step_failures`: `6`
- `accounting.pricing_version`: `v1`
- `accounting.fallback_pricing`: empty map (no built-in provider/model pricing
  entries)

## Validation Rules

- Exactly one of `prompt` and `prompt_template` MUST be set.
- Exactly one of `context` and `context_template` MUST be set.
- `llm.provider` MUST be set after merge.
- `llm.model` MUST be set after merge.
- `llm.gateway` MUST be `openrouter` in this initial release.
- `llm.openrouter` MAY be omitted when `llm.gateway` is `openrouter`; effective
  OpenRouter fields MUST resolve from merge precedence (`env > file > defaults`).
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
- `llm.reasoning.effort` MUST be one of:
  - `minimal`
  - `low`
  - `medium`
  - `high`
- If `llm.reasoning.enabled=false`, `llm.reasoning.effort` MAY be present and
  is ignored for inference request construction.
- `guardrails.max_steps_per_node` MUST be `>= 1`.
- `guardrails.max_total_steps_per_run` MUST be `>= 1`.
- `guardrails.max_run_duration_ms` MUST be `>= 1`.
- `guardrails.max_consecutive_step_failures` MUST be `>= 1`.
- `accounting.pricing_version` MUST be non-empty after merge.
- `accounting.fallback_pricing` MAY be omitted entirely.
- When `accounting.fallback_pricing.<provider>.<model>` is present,
  `input_microusd_per_million_tokens` MUST be `>= 1`.
- When `accounting.fallback_pricing.<provider>.<model>` is present,
  `output_microusd_per_million_tokens` MUST be `>= 1`.
- When `accounting.fallback_pricing.<provider>.<model>.reasoning_microusd_per_million_tokens`
  is present, it MUST be `>= 1`.

## Environment Override Contract

Viper runtime environment configuration MUST be initialized with:

- `SetEnvPrefix("SIGIL_RUN")`
- `SetEnvKeyReplacer(strings.NewReplacer(".", "_"))`
- `AutomaticEnv()`

The following environment variables are representative contract examples:

- `SIGIL_RUN_LLM_PROVIDER`
- `SIGIL_RUN_LLM_MODEL`
- `SIGIL_RUN_LLM_GATEWAY`
- `SIGIL_RUN_LLM_REASONING_ENABLED`
- `SIGIL_RUN_LLM_REASONING_EFFORT`
- `SIGIL_RUN_LLM_OPENROUTER_BASE_URL`
- `SIGIL_RUN_LLM_OPENROUTER_REQUEST_TIMEOUT_MS`
- `SIGIL_RUN_LLM_OPENROUTER_API_KEY_ENV`
- `SIGIL_RUN_RLM_ENABLED`
- `SIGIL_RUN_RLM_MAX_DEPTH`
- `SIGIL_RUN_GUARDRAILS_MAX_STEPS_PER_NODE`
- `SIGIL_RUN_GUARDRAILS_MAX_TOTAL_STEPS_PER_RUN`
- `SIGIL_RUN_GUARDRAILS_MAX_RUN_DURATION_MS`
- `SIGIL_RUN_GUARDRAILS_MAX_CONSECUTIVE_STEP_FAILURES`
- `SIGIL_RUN_ACCOUNTING_PRICING_VERSION`
- `SIGIL_RUN_ACCOUNTING_FALLBACK_PRICING_OPENAI_GPT_5_1_INPUT_MICROUSD_PER_MILLION_TOKENS`
- `SIGIL_RUN_ACCOUNTING_FALLBACK_PRICING_OPENAI_GPT_5_1_OUTPUT_MICROUSD_PER_MILLION_TOKENS`
- `SIGIL_RUN_ACCOUNTING_FALLBACK_PRICING_OPENAI_GPT_5_1_REASONING_MICROUSD_PER_MILLION_TOKENS`
- `SIGIL_RUN_SYSTEM_PROMPT_APPEND`
- `SIGIL_RUN_PROMPT`
- `SIGIL_RUN_PROMPT_TEMPLATE`
- `SIGIL_RUN_CONTEXT`
- `SIGIL_RUN_CONTEXT_TEMPLATE`

## Implementation Constraints

- Configuration types MUST be defined in `internal/config/types.go`.
- Config fields MUST use `yaml` struct tags.
- Nested YAML keys MUST be represented using nested structs in `types.go`.
- Default values MUST be defined in `internal/config/defaults.go`.
- Bootstrap and merge orchestration MUST use Viper and enforce the precedence
  and validation rules defined in this PRD.

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

Given merged run configuration missing `llm.provider` or `llm.model`  
When validation runs  
Then initialization fails with a required-field validation error.

### Scenario SCN-0003: Requires exactly one of prompt and prompt_template

Given merged run configuration with both or neither of `prompt` and
`prompt_template`  
When validation runs  
Then initialization fails with an exclusivity validation error.

### Scenario SCN-0004: Requires exactly one of context and context_template

Given merged run configuration with both or neither of `context` and
`context_template`  
When validation runs  
Then initialization fails with an exclusivity validation error.

### Scenario SCN-0005: Defaults llm.gateway to openrouter when omitted

Given merged run configuration omits `llm.gateway`  
When defaults are applied  
Then `llm.gateway` resolves to `openrouter`.

### Scenario SCN-0006: Allows omitted llm.openrouter block when gateway is openrouter

Given merged run configuration where `llm.gateway` is `openrouter` and
`llm.openrouter` is absent  
When defaults are applied and validation runs  
Then initialization succeeds and OpenRouter fields resolve from merged values.

### Scenario SCN-0007: Rejects unsupported llm.gateway values

Given merged run configuration where `llm.gateway` is not `openrouter`  
When validation runs  
Then initialization fails with an unsupported-gateway validation error.

### Scenario SCN-0008: Applies OpenRouter defaults when openrouter fields are omitted

Given merged run configuration with `llm.gateway=openrouter`, `llm.openrouter`
present, and one or more missing `llm.openrouter` leaf values  
When defaults are applied  
Then `base_url`, `request_timeout_ms`, and `api_key_env` resolve to their
defined defaults.

### Scenario SCN-0009: Applies RLM defaults when rlm fields are omitted

Given merged run configuration omits `rlm.enabled` and `rlm.max_depth`  
When defaults are applied  
Then `rlm.enabled=true` and `rlm.max_depth=3`.

### Scenario SCN-0010: Accepts required values provided entirely by environment variables

Given `sigil-run.yaml` is missing and required fields are set via
`SIGIL_RUN_*` environment variables  
When run configuration initializes  
Then merged run configuration is valid and initialization succeeds.

### Scenario SCN-0011: Accepts llm.provider only when value is openai or anthropic

Given merged run configuration with `llm.provider` set  
When validation runs  
Then validation succeeds only for `openai` or `anthropic`.

### Scenario SCN-0012: Validates llm.model against allowed list for provider openai

Given merged run configuration with `llm.provider=openai`  
When validation runs  
Then `llm.model` must be in the allowed OpenAI model set.

### Scenario SCN-0013: Validates llm.model against allowed list for provider anthropic

Given merged run configuration with `llm.provider=anthropic`  
When validation runs  
Then `llm.model` must be in the allowed Anthropic model set.

### Scenario SCN-0014: Rejects run configuration when llm.model is not allowed for selected llm.provider

Given merged run configuration with provider/model pair outside allowed mapping  
When validation runs  
Then initialization fails with unsupported provider-model validation error.

### Scenario SCN-0015: Applies default reasoning config values when llm.reasoning block is omitted

Given merged run configuration omits `llm.reasoning` block  
When defaults are applied  
Then `llm.reasoning.enabled=true` and `llm.reasoning.effort=medium`.

### Scenario SCN-0016: Accepts llm.reasoning.effort only when value is minimal low medium or high

Given merged run configuration with `llm.reasoning.effort` set  
When validation runs  
Then validation succeeds only for `minimal`, `low`, `medium`, or `high`.

### Scenario SCN-0017: Allows reasoning effort to be present and ignored when llm.reasoning.enabled is false

Given merged run configuration where `llm.reasoning.enabled=false` and effort is
set  
When validation and request-construction semantics are applied  
Then configuration is valid and effort is ignored for inference request
construction.

### Scenario SCN-0018: Applies SIGIL_RUN environment overrides for llm.reasoning.enabled and llm.reasoning.effort

Given run configuration values and corresponding reasoning environment variables  
When merge precedence is applied  
Then `SIGIL_RUN_LLM_REASONING_ENABLED` and
`SIGIL_RUN_LLM_REASONING_EFFORT` override file/default values.

### Scenario SCN-0019: Defines deterministic guardrails section in run config schema

Given run configuration schema definitions are loaded  
When schema fields are inspected  
Then guardrails keys exist for max_steps_per_node max_total_steps_per_run max_run_duration_ms and max_consecutive_step_failures.

### Scenario SCN-0020: Applies deterministic guardrail defaults when guardrails fields are omitted

Given merged run configuration omits guardrails fields  
When defaults are applied  
Then guardrail fields resolve to deterministic default values.

### Scenario SCN-0021: Applies SIGIL_RUN environment overrides for guardrails fields

Given run configuration values and corresponding guardrails environment variables  
When merge precedence is applied  
Then SIGIL_RUN guardrail environment values override file/default values.

### Scenario SCN-0022: Rejects run configuration when deterministic guardrail values are non-positive

Given merged run configuration with one or more non-positive guardrail values  
When validation runs  
Then initialization fails with guardrail validation error.

### Scenario SCN-0023: Defines accounting section in run config schema

Given run configuration contract definitions are loaded  
When accounting configuration keys are inspected  
Then `accounting.pricing_version` and `accounting.fallback_pricing` are
defined in the run config schema.

### Scenario SCN-0024: Applies default accounting pricing version when accounting section is omitted

Given merged run configuration omits the `accounting` section  
When defaults are applied  
Then `accounting.pricing_version` resolves to `v1`.

### Scenario SCN-0025: Applies SIGIL_RUN environment overrides for accounting fallback pricing

Given run configuration values and corresponding accounting environment
variables  
When merge precedence is applied  
Then SIGIL_RUN accounting environment values override file and default values.

### Scenario SCN-0026: Rejects run configuration when accounting fallback pricing values are non-positive

Given merged run configuration with one or more non-positive accounting
fallback pricing values  
When validation runs  
Then initialization fails with accounting fallback-pricing validation error.
