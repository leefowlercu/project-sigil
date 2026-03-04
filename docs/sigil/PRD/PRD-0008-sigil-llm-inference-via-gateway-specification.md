# PRD-0008: Sigil LLM Inference via Gateway Specification

## Status

Accepted

## Context

`sigil` requires a deterministic inference behavior contract that:

- Uses gateway-agnostic orchestration at harness core.
- Uses OpenRouter as the initial concrete gateway.
- Enforces strict structured output validation for downstream runtime logic.
- Supports reasoning-enabled inference for model behavior quality.

This PRD defines inference behavior contracts only and does not define runtime
implementation details.

## Goals

- Define gateway resolution and abstraction behavior for inference calls.
- Define OpenRouter request construction behavior for v1.
- Define strict schema registry usage and structured-output behavior.
- Define mandatory healing and reasoning behavior.
- Define deterministic retry and failure behavior.
- Define canonical normalized successful inference output shape.

## Non-Goals

- Defining app-server protocol behavior.
- Defining gateway implementations beyond OpenRouter in v1.
- Defining streaming inference behavior in v1.
- Defining storage/projection contracts for inference results.

## Gateway Abstraction Contract

- Harness inference core MUST call inference through a gateway interface.
- Gateway implementations MUST be resolved through registry lookup by
  `llm.gateway`.
- Gateway adapters MUST isolate gateway-specific transport mapping.
- Harness inference core MUST NOT depend on OpenRouter transport-specific types.

## Gateway Resolution Contract

- In v1, `llm.gateway` MUST resolve to `openrouter`.
- Missing/unknown gateway registry key MUST fail inference initialization with a
  typed gateway resolution error.

## OpenRouter Request Construction Contract

- Requests MUST use OpenRouter Responses API.
- Requests MUST be non-streaming in v1.
- Requests MUST provide model input as ordered role-based message arrays.
- Message order MUST preserve harness ordering semantics (`system` then `user`)
  per step.
- Requests MUST use `json_schema` strict structured outputs.
- Requests MUST resolve schema definitions from central registry by `schema_id`.
- Requests MUST enable response healing plugin.
- Requests MUST apply reasoning only when `llm.reasoning.enabled=true`.
- Requests MUST omit reasoning block when `llm.reasoning.enabled=false`.

## Central Schema Registry Contract

- Inference schema resolution MUST use central registry only.
- Inline ad-hoc schema definitions are out-of-contract in v1.
- v1 required schema IDs:
  - `sigil.rlm.response.v1`
  - `sigil.llm.answer.v1`
- Unresolved `schema_id` MUST fail request construction with typed schema lookup
  error.

## Structured Response Schema Contract (`sigil.rlm.response.v1`)

`sigil.rlm.response.v1` defines the required shape for
`validated_payload` in normalized inference output.

Top-level fields:

- `decision` (required string enum): `continue`, `final`
- `continuation` (optional object):
  - `repl_code` (required string, non-empty Go code when `continuation`
    present)
  - `intent` (required string, non-empty when `continuation` present)
  - `expected_observation` (required string, non-empty when `continuation`
    present)
- `final` (optional object):
  - `answer` (required string, non-empty when `final` present)
  - `evidence` (required array with at least one item when `final` present)
  - `confidence` (optional enum: `low`, `medium`, `high`)

Evidence item fields:

- `ref` (required string)
- `chunk_id` (optional string)
- `span_start` (optional integer >= 0)
- `span_end` (optional integer >= 0 and `>= span_start` when both exist)

Schema invariants:

- `decision=continue` MUST include `continuation.repl_code` and MUST NOT
  include `final`.
- `decision=continue` MUST include `continuation.intent` and
  `continuation.expected_observation`.
- `decision=final` MUST include `final` and MUST NOT include `continuation`.
- `decision=final` MUST include `final.evidence` with at least one item.
- `final.confidence` MUST be omitted or one of `low|medium|high`.
- Unknown top-level fields or unknown nested fields MUST fail strict schema
  validation.
- Markdown code-fence conventions (for example `repl-go`) are out-of-contract
  for structured payload validation in v1.

## Structured Response Schema Contract (`sigil.llm.answer.v1`)

`sigil.llm.answer.v1` defines strict plain-subcall output shape for one-shot
LM subcalls.

Top-level fields:

- `answer` (required string, non-empty)

Schema invariants:

- Payload MUST include `answer`.
- `answer` MUST be a non-empty string.
- Unknown top-level fields MUST fail strict schema validation.

## Structured Output Contract

- All inference requests MUST require strict schema validation.
- Successful inference responses MUST validate against requested schema.
- Non-conforming structured responses MUST be treated as inference failure.

## Response Healing Contract

- Response healing plugin MUST be enabled for all requests.
- Healing is mandatory and non-optional in v1.
- If healed response still fails strict schema validation, inference MUST fail
  with typed output-validation error.

## Reasoning Contract

- `llm.reasoning.enabled=true` MUST include reasoning config with configured
  effort in request payload.
- `llm.reasoning.effort` MUST be one of:
  - `minimal`
  - `low`
  - `medium`
  - `high`
- `llm.reasoning.enabled=false` MUST omit reasoning config from request payload.
- Plain-subcall cheap path requests MAY force reasoning disabled behavior even
  when top-level run config has reasoning enabled.
- Normalized output MUST include a dedicated top-level `reasoning` key.
- `reasoning.enabled` MUST reflect effective request behavior.
- `reasoning.effort` MUST reflect configured effort when enabled; it MUST be
  `null` when disabled.
- Provider reasoning token accounting MUST be mapped under
  `usage.reasoning_tokens` when available.
- Provider reasoning artifacts (for example summaries or encrypted reasoning
  content references) MUST be mapped under top-level `reasoning` and MUST NOT
  be represented only in `raw_metadata`.
- If configured model/provider pair does not support required reasoning behavior
  at runtime, inference MUST fail with typed reasoning capability error.

## Retry and Failure Contract

- Transient retry policy applies only to HTTP 429 and HTTP 5xx responses.
- Retry policy MUST be bounded to 3 total attempts:
  - Attempt 1 initial call
  - Up to 2 retries
- Backoff MUST be exponential with:
  - base delay `250ms`
  - jitter enabled
  - max delay cap `2s`
- After retries are exhausted, inference MUST fail with typed gateway failure
  error.

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

The following are explicitly deferred:

- Streaming inference behavior.
- Multi-gateway runtime behavior and selection policies.
- Cross-version output migration mechanics.
- Advanced routing/fallback heuristics across models/providers.

## Acceptance Scenarios

### Scenario SCN-0000: Resolves inference gateway through registry using llm.gateway

Given configured `llm.gateway` value  
When inference initialization resolves a gateway  
Then resolution occurs through gateway registry lookup.

### Scenario SCN-0001: Uses OpenRouter Responses API in non-streaming mode for v1

Given configured OpenRouter gateway  
When inference request is constructed  
Then request targets OpenRouter Responses API in non-streaming mode.

### Scenario SCN-0002: Resolves schema_id sigil.rlm.response.v1 from central registry for inference requests

Given inference request with `schema_id=sigil.rlm.response.v1`  
When request construction runs  
Then schema is resolved from central registry and applied to request.

### Scenario SCN-0003: Uses schema_id sigil.rlm.response.v1 for terminal inference responses

Given terminal inference response request  
When request construction runs  
Then `schema_id=sigil.rlm.response.v1` is resolved from central registry and
applied to request.

### Scenario SCN-0004: Requires strict json_schema structured outputs on all inference requests

Given any inference request  
When request payload is constructed  
Then strict `json_schema` structured output mode is required.

### Scenario SCN-0005: Enables response healing plugin on all inference requests

Given any inference request  
When request payload is constructed  
Then response healing plugin is enabled.

### Scenario SCN-0006: Applies reasoning configuration when llm.reasoning.enabled is true

Given `llm.reasoning.enabled=true` and valid reasoning effort  
When request payload is constructed  
Then reasoning config is included using configured effort.

### Scenario SCN-0007: Omits reasoning request block when llm.reasoning.enabled is false

Given `llm.reasoning.enabled=false`  
When request payload is constructed  
Then reasoning config is omitted.

### Scenario SCN-0008: Retries up to three total attempts with exponential backoff on 429 and 5xx responses

Given transient gateway response status 429 or 5xx  
When inference is executed  
Then runtime retries with bounded policy (3 total attempts, exponential backoff,
base 250ms, jitter, max 2s delay).

### Scenario SCN-0009: Fails with typed inference error after bounded retries are exhausted

Given transient 429/5xx responses across all allowed attempts  
When retry budget is exhausted  
Then inference fails with typed gateway failure error.

### Scenario SCN-0010: Fails with typed inference error when healed response does not satisfy strict schema

Given healing-enabled inference response  
When final response does not satisfy strict schema validation  
Then inference fails with typed output-validation error.

### Scenario SCN-0011: Fails with typed inference error when configured provider model pair does not support required reasoning behavior

Given reasoning-enabled inference for unsupported provider/model runtime behavior  
When inference is executed  
Then inference fails with typed reasoning capability error.

### Scenario SCN-0012: Rejects inference request when schema_id is not found in central registry

Given inference request with unknown schema ID  
When request construction resolves schema  
Then request is rejected with typed schema lookup error.

### Scenario SCN-0013: Returns canonical normalized inference response shape on success

Given successful inference execution  
When normalized output is emitted  
Then output contains all required canonical fields.

### Scenario SCN-0014: Requires decision discriminator values continue or final in sigil.rlm.response.v1

Given inference response payload for `sigil.rlm.response.v1`  
When strict schema validation runs  
Then `decision` is required and value is either `continue` or `final`.

### Scenario SCN-0015: Requires continuation repl_code and forbids final branch when decision is continue

Given inference response payload with `decision=continue`  
When strict schema validation runs  
Then `continuation.repl_code` is required and `final` is absent.

### Scenario SCN-0016: Requires final branch and forbids continuation branch when decision is final

Given inference response payload with `decision=final`  
When strict schema validation runs  
Then `final.answer` is required and `continuation` is absent.

### Scenario SCN-0017: Rejects unknown fields in sigil.rlm.response.v1 payloads

Given inference response payload containing unknown fields  
When strict schema validation runs  
Then validation fails with typed output-validation error.

### Scenario SCN-0018: Emits reasoning data under top-level reasoning key and reasoning token counts under usage

Given reasoning-enabled inference where gateway returns reasoning artifacts and
reasoning token counts  
When normalized output is emitted  
Then reasoning artifacts are present under top-level `reasoning` and reasoning
token counts are present under `usage.reasoning_tokens`.

### Scenario SCN-0019: Resolves schema_id sigil.llm.answer.v1 from central registry for plain subcall inference requests

Given inference request with `schema_id=sigil.llm.answer.v1`  
When request construction resolves schema  
Then schema is resolved from central registry and applied to request.

### Scenario SCN-0020: Requires non-empty answer field in sigil.llm.answer.v1 payloads

Given inference response payload for `sigil.llm.answer.v1`  
When strict schema validation runs  
Then payload is valid only when `answer` is present and non-empty.

### Scenario SCN-0021: Constructs plain-subcall inference requests as ordered message arrays

Given plain subcall inference request input  
When OpenRouter request payload is constructed  
Then request uses ordered role-based message arrays preserving role order.

### Scenario SCN-0022: Supports cheap plain-subcall path with reasoning omitted from request payload

Given plain subcall cheap-path inference execution  
When request payload is constructed  
Then reasoning block is omitted from request payload.

### Scenario SCN-0023: Requires continuation intent and expected_observation fields when decision is continue

Given inference response payload with `decision=continue`  
When strict schema validation runs  
Then `continuation.intent` and `continuation.expected_observation` are required
and non-empty.

### Scenario SCN-0024: Requires final evidence array when decision is final

Given inference response payload with `decision=final`  
When strict schema validation runs  
Then `final.evidence` is required with at least one evidence item.

### Scenario SCN-0025: Restricts final confidence to enum low medium or high when present

Given inference response payload with `decision=final` and `final.confidence`  
When strict schema validation runs  
Then `final.confidence` is valid only when value is one of low medium or high.
