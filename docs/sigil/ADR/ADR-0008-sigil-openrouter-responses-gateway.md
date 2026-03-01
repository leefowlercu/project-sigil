# ADR-0008: Sigil OpenRouter Responses Gateway

## Status

Accepted

## Date

2026-03-01

## Context

`sigil` needs an initial concrete gateway implementation for LLM inference that
aligns with the gateway abstraction baseline and supports strict structured
outputs for harness behavior contracts.

For v1, the project intentionally limits runtime gateway scope while preserving
future gateway extensibility through ADR-0007.

## Decision

`sigil` v1 inference gateway MUST be OpenRouter using the OpenRouter Responses
API with strict structured outputs, mandatory response healing, and mandatory
reasoning support.

## Decision Details

- OpenRouter is the only supported `llm.gateway` in v1.
- Inference requests MUST use OpenRouter Responses API.
- Inference requests MUST use strict `json_schema` structured outputs.
- Response healing plugin MUST be enabled for all inference requests.
- Inference mode MUST be non-streaming in v1.
- OpenRouter reasoning configuration MUST be supported through run config keys:
  - `llm.reasoning.enabled`
  - `llm.reasoning.effort`

## Alternatives Considered

- Support multiple gateways in v1: rejected to reduce initial integration
  complexity and keep baseline behavior deterministic.
- Streaming-first v1 inference: rejected because response healing support is
  aligned with non-streaming usage.
- Optional healing/structured outputs in v1: rejected to avoid divergent output
  contracts and reduce downstream validation uncertainty.

## Consequences

- Gateway behavior is deterministic for v1 and testable via strict contracts.
- Future gateway additions remain possible through ADR-0007 abstractions.
- Streaming inference support is deferred.

## Migration/Adoption Notes

- Any change to v1 OpenRouter-only gateway scope requires superseding ADR.
- Any move to optional structured output/healing behavior requires PRD contract
  updates and mapped acceptance changes.
- Cross-version inference response migration rules are deferred.

## Related Documents

- [Sigil ADR Index](README.md)
- [ADR-0007 LLM Gateway Abstraction](ADR-0007-sigil-llm-gateway-abstraction.md)
- [PRD-0003 Run Configuration](../PRD/PRD-0003-sigil-run-config-specification.md)
- [PRD-0008 LLM Inference via Gateway Specification](../PRD/PRD-0008-sigil-llm-inference-via-gateway-specification.md)
- [OpenRouter Responses API](https://openrouter.ai/docs/api-reference/responses/overview)
- [OpenRouter Structured Outputs](https://openrouter.ai/docs/guides/structured-outputs)
- [OpenRouter Response Healing](https://openrouter.ai/docs/features/plugins/response-healing)
- [OpenRouter Responses Reasoning](https://openrouter.ai/docs/api-reference/responses/reasoning)
