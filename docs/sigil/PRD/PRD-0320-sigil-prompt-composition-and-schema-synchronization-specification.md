# PRD-0320: Sigil Prompt Composition and Schema Synchronization Specification

## Status

Draft

## Context

`sigil` resolves provider-specific base prompts, optional prompt append text,
and runtime schema instructions before each model step. Those rules need one
normative owner so prompt behavior does not drift from strict schema
validation.

This PRD owns:

- provider base prompt selection
- fallback prompt behavior
- `system_prompt_append` composition
- runtime schema-text rendering from the same registry used for validation

## Goals

- Define deterministic provider base prompt resolution.
- Define composition rules for `system_prompt_append`.
- Define prompt-schema synchronization using the central schema registry.
- Keep prompt rendering and strict validation mechanically aligned.

## Non-Goals

- Defining prompt text content revisions or authoring workflow.
- Defining remote prompt registries or version negotiation.
- Defining inference transport behavior or response normalization.
- Defining end-user prompt templating beyond `system_prompt_append`.

## Provider Base Prompt Contract

- Runtime MUST resolve one base system prompt from a hard-coded provider map.
- Required hard-coded entries are:
  - `openai`
  - `anthropic`
- If a provider has no dedicated prompt entry, runtime MUST fall back to the OpenAI base prompt.
- Resolved prompt selection MUST be deterministic for equivalent provider input.

## System Prompt Append Contract

- `system_prompt_append` MUST default to the empty string.
- If `system_prompt_append` is empty after trim, the effective system prompt MUST be the resolved base prompt.
- If `system_prompt_append` is non-empty after trim, the effective system prompt MUST be:
  1. the resolved base prompt
  2. two newline characters
  3. the raw configured append text

## Runtime Schema Synchronization Contract

- Provider base prompts MUST live in `sigil/internal/harness/system_prompts.go`.
- The system-prompt schema block MUST be rendered at runtime from the central registry definition for `sigil.rlm.response.v1`.
- Effective prompt resolution occurs after runtime schema rendering.
- Prompt instructions and strict inference validation MUST derive from the same registry definition.

## Effective Prompt Composition Order

For each model step, effective system prompt composition MUST be:

1. resolve provider base prompt
2. render schema block from the central registry
3. compose base prompt and rendered schema block
4. apply `system_prompt_append`

## Deferred Contracts

- Dynamic prompt registries sourced from files or remote stores
- provider-specific prompt versioning and migration workflows
- prompt variants by execution profile or scenario

## Acceptance Scenarios

### Scenario SCN-0000: Resolves OpenAI base system prompt when llm.provider is openai

Given run configuration with `llm.provider=openai`  
When runtime resolves the base system prompt  
Then the OpenAI base prompt is selected.

### Scenario SCN-0001: Resolves Anthropic base system prompt when llm.provider is anthropic

Given run configuration with `llm.provider=anthropic`  
When runtime resolves the base system prompt  
Then the Anthropic base prompt is selected.

### Scenario SCN-0002: Falls back to OpenAI base system prompt when provider-specific prompt is not registered

Given run configuration with a provider value lacking a dedicated prompt mapping  
When runtime resolves the base system prompt  
Then the OpenAI base prompt is selected as fallback.

### Scenario SCN-0003: Appends system_prompt_append to resolved base system prompt when append is non-empty

Given resolved base system prompt and non-empty `system_prompt_append`  
When effective prompt composition runs  
Then the append text is added after the base prompt with two newline characters.

### Scenario SCN-0004: Uses resolved base system prompt unchanged when system_prompt_append is empty

Given resolved base system prompt and empty `system_prompt_append`  
When effective prompt composition runs  
Then the effective system prompt matches the resolved base prompt.

### Scenario SCN-0005: Generates system prompt schema block from central registry definition sigil.rlm.response.v1 at runtime

Given runtime prompt rendering begins  
When the schema block is generated  
Then it is rendered from the central registry definition for `sigil.rlm.response.v1`.

### Scenario SCN-0006: Applies provider prompt resolution and system_prompt_append composition after runtime schema rendering

Given runtime schema rendering has completed  
When effective prompt composition runs  
Then provider prompt resolution and `system_prompt_append` composition occur after schema rendering.

### Scenario SCN-0007: Maintains prompt-schema parity by deriving prompt schema text from the same registry definition used for strict inference validation

Given prompt rendering and strict validation both depend on `sigil.rlm.response.v1`  
When prompt-schema parity is evaluated  
Then both surfaces derive from the same registry definition.
