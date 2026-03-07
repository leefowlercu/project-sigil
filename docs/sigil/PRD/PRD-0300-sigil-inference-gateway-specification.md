# PRD-0300: Sigil Inference Gateway Specification

## Status

Accepted

## Context

`sigil` requires a deterministic inference contract that isolates gateway
transport behavior from the harness and produces one normalized response shape
for downstream runtime logic.

This PRD owns:

- gateway abstraction and resolution
- request construction rules
- reasoning configuration and request behavior
- plain-subcall adapter behavior
- retry and failure behavior
- normalized successful inference output

Structured response schemas and evidence semantics are defined in `PRD-0310`.
Prompt composition and runtime schema-text rendering are defined in `PRD-0320`.

## Goals

- Define gateway resolution and abstraction behavior for inference calls.
- Define OpenRouter transport behavior for the initial release.
- Define reasoning configuration defaults and request behavior.
- Define retry, healing, and normalization behavior.
- Define plain-subcall fallback behavior at the adapter boundary.

## Non-Goals

- Defining event persistence or artifact storage contracts.
- Defining final-answer evidence validation behavior.
- Defining prompt-composition ownership beyond request inputs.
- Defining streaming inference behavior in this release.

## Gateway Abstraction Contract

- Harness inference core MUST call inference through a gateway interface.
- Gateway implementations MUST be resolved through registry lookup by `llm.gateway`.
- Gateway adapters MUST isolate gateway-specific transport mapping.
- Harness inference core MUST NOT depend on OpenRouter transport-specific types.

## Gateway Resolution Contract

- In this release, `llm.gateway` MUST resolve to `openrouter`.
- Missing or unknown gateway registry keys MUST fail inference initialization with a typed gateway-resolution error.

## Request Construction Contract

- Requests MUST use the OpenRouter Responses API.
- Requests MUST be non-streaming in this release.
- Requests MUST provide model input as ordered role-based message arrays.
- Requests MUST use strict `json_schema` structured outputs.
- Requests MUST resolve schema definitions from the central registry by `schema_id`.
- Inline ad-hoc schema definitions are out of contract.
- Requests MUST enable the response-healing plugin.

## Reasoning Configuration Contract

- `llm.reasoning.enabled` defaults to `true`.
- `llm.reasoning.effort` defaults to `medium`.
- `llm.reasoning.effort` MUST be one of:
  - `minimal`
  - `low`
  - `medium`
  - `high`
- If `llm.reasoning.enabled=false`, `llm.reasoning.effort` MAY be present and is ignored for request construction.
- `SIGIL_RUN_LLM_REASONING_ENABLED` and `SIGIL_RUN_LLM_REASONING_EFFORT` MUST override file values through normal run-config merge precedence.

## Reasoning Request and Output Contract

- `llm.reasoning.enabled=true` MUST include reasoning config with the configured effort in the request payload.
- `llm.reasoning.enabled=false` MUST omit reasoning config from the request payload.
- Plain-subcall cheap-path requests MAY force reasoning-disabled behavior even when the top-level run config has reasoning enabled.
- Normalized output MUST include a dedicated top-level `reasoning` key.
- `reasoning.enabled` MUST reflect effective request behavior.
- `reasoning.effort` MUST reflect configured effort when enabled and MUST be `null` when disabled.
- Provider reasoning token accounting MUST be mapped under `usage.reasoning_tokens` when available.
- Provider reasoning artifacts MUST be mapped under top-level `reasoning` and MUST NOT be represented only in `raw_metadata`.
- If the configured provider or model pair does not support required reasoning behavior at runtime, inference MUST fail with a typed reasoning-capability error.

## Plain-Subcall Inference Contract

- Plain-subcall requests MUST use the schema ID defined for plain-subcall answers in `PRD-0310`.
- Plain-subcall requests MUST be constructed as ordered message arrays.
- Plain-subcall adapter behavior MUST preserve the configured provider and model routing from the active node.

## Plain-Subcall Extraction Fallback Contract

- Raw-text fallback is schema-scoped and applies only to `sigil.llm.answer.v1`.
- If strict extraction fails for `sigil.llm.answer.v1` and non-empty raw output text is available, the adapter MUST normalize payload to:
  - `{"answer":"<trimmed raw text>"}`
- For any schema other than `sigil.llm.answer.v1`, raw-text extraction fallback is forbidden.
- When fallback is used, normalized output `raw_metadata` MUST include extraction mode and source metadata.

## Retry and Failure Contract

- Transient retry policy applies only to HTTP `429` and HTTP `5xx` responses.
- Retry policy MUST be bounded to 3 total attempts:
  1. initial call
  2. first retry
  3. final retry
- Backoff MUST be exponential with:
  - base delay `250ms`
  - jitter enabled
  - max delay cap `2s`
- If the healed response still fails strict schema validation, inference MUST fail with typed output-validation behavior.
- After retries are exhausted, inference MUST fail with a typed gateway-failure error.

## Canonical Output Contract

On successful inference, normalized output MUST include:

- `schema_id`
- `validated_payload`
- `gateway`
- `provider`
- `model`
- `gateway_response_id`
- `usage`
- `reasoning`
- `finish_status`
- `raw_metadata`

## Deferred Contracts

- Streaming inference behavior
- multi-gateway runtime behavior and selection policies
- cross-version output migration mechanics
- advanced routing or fallback heuristics across models and providers

## Acceptance Scenarios

### Scenario SCN-0000: Applies default reasoning config values when llm.reasoning block is omitted

Given run configuration omits `llm.reasoning`  
When defaults are applied  
Then `llm.reasoning.enabled=true` and `llm.reasoning.effort=medium`.

### Scenario SCN-0001: Accepts llm.reasoning.effort only when value is minimal low medium or high

Given run configuration sets `llm.reasoning.effort`  
When validation runs  
Then only `minimal`, `low`, `medium`, and `high` are accepted.

### Scenario SCN-0002: Allows reasoning effort to be present and ignored when llm.reasoning.enabled is false

Given run configuration sets `llm.reasoning.enabled=false`  
And `llm.reasoning.effort` is present  
When inference request construction runs  
Then the effort value is accepted but omitted from effective request behavior.

### Scenario SCN-0003: Applies SIGIL_RUN environment overrides for llm.reasoning.enabled and llm.reasoning.effort

Given file configuration defines `llm.reasoning` values  
And corresponding `SIGIL_RUN_LLM_REASONING_*` environment variables are present  
When run configuration is merged  
Then environment values override file values.

### Scenario SCN-0004: Resolves inference gateway through registry using llm.gateway

Given configured `llm.gateway` value  
When inference initialization resolves a gateway  
Then resolution occurs through gateway registry lookup.

### Scenario SCN-0005: Uses OpenRouter Responses API in non-streaming mode for v1

Given a valid inference request  
When the OpenRouter adapter constructs transport behavior  
Then it uses the Responses API in non-streaming mode.

### Scenario SCN-0006: Requires strict json_schema structured outputs on all inference requests

Given an inference request  
When transport payload is constructed  
Then strict `json_schema` structured outputs are required.

### Scenario SCN-0007: Enables response healing plugin on all inference requests

Given an inference request  
When transport payload is constructed  
Then the response-healing plugin is enabled.

### Scenario SCN-0008: Applies reasoning configuration when llm.reasoning.enabled is true

Given `llm.reasoning.enabled=true` and a valid effort value  
When the request payload is constructed  
Then reasoning config is included using the configured effort.

### Scenario SCN-0009: Omits reasoning request block when llm.reasoning.enabled is false

Given `llm.reasoning.enabled=false`  
When the request payload is constructed  
Then reasoning config is omitted.

### Scenario SCN-0010: Retries up to three total attempts with exponential backoff on 429 and 5xx responses

Given an inference call returns HTTP `429` or `5xx` responses  
When retry policy executes  
Then inference retries up to three total attempts using exponential backoff with jitter.

### Scenario SCN-0011: Fails with typed inference error after bounded retries are exhausted

Given retryable inference failures continue through the final allowed attempt  
When retry policy completes  
Then inference fails with a typed gateway-failure error.

### Scenario SCN-0012: Fails with typed inference error when healed response does not satisfy strict schema

Given response healing runs  
And the healed response still violates the requested strict schema  
When response validation completes  
Then inference fails with typed output-validation behavior.

### Scenario SCN-0013: Fails with typed inference error when configured provider model pair does not support required reasoning behavior

Given reasoning-enabled inference for unsupported provider or model runtime behavior  
When request execution begins  
Then inference fails with typed reasoning-capability behavior.

### Scenario SCN-0014: Returns canonical normalized inference response shape on success

Given a successful inference response  
When normalization completes  
Then the normalized output includes the canonical top-level fields defined by this PRD.

### Scenario SCN-0015: Emits reasoning data under top-level reasoning key and reasoning token counts under usage

Given reasoning-enabled inference returns reasoning artifacts and reasoning token counts  
When output normalization completes  
Then reasoning artifacts are present under top-level `reasoning` and reasoning token counts are present under `usage.reasoning_tokens`.

### Scenario SCN-0016: Constructs plain-subcall inference requests as ordered message arrays

Given a plain-subcall inference request  
When the adapter constructs the transport payload  
Then model input is encoded as an ordered message array.

### Scenario SCN-0017: Supports cheap plain-subcall path with reasoning omitted from request payload

Given plain-subcall execution chooses the cheap path  
When the adapter constructs the request payload  
Then the reasoning block is omitted from the request payload.

### Scenario SCN-0018: Applies schema-specific raw-text fallback for sigil.llm.answer.v1 extraction failures

Given response extraction fails for schema `sigil.llm.answer.v1`  
And non-empty raw output text is available  
When fallback normalization runs  
Then the adapter emits `{"answer":"<trimmed raw text>"}`.

### Scenario SCN-0019: Rejects raw-text fallback for non sigil.llm.answer.v1 schemas

Given response extraction fails for a schema other than `sigil.llm.answer.v1`  
When fallback policy is evaluated  
Then raw-text fallback is rejected.

### Scenario SCN-0020: Emits extraction-mode metadata in raw_metadata when plain-subcall fallback is applied

Given the plain-subcall extraction fallback path is used  
When output normalization completes  
Then normalized `raw_metadata` records extraction mode and source metadata.
