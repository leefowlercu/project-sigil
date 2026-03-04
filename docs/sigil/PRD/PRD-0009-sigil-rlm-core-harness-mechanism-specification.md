# PRD-0009: Sigil RLM Core Harness Mechanism Specification

## Status

Draft

## Context

`sigil` already defines run lifecycle (`PRD-0006`), event envelopes (`PRD-0007`),
and gateway inference contracts (`PRD-0008`).

What is still missing is a single behavioral contract for the harness mechanism
that executes recursive language-model control steps, integrates a Go REPL
execution environment, and deterministically resolves provider-specific system
prompts.

This PRD defines that harness mechanism contract for v1.

## Goals

- Define the v1 RLM harness control loop for root and recursive nodes.
- Define deterministic system prompt resolution by `llm.provider`.
- Define fallback behavior for providers that do not have dedicated prompt
  entries.
- Define how `system_prompt_append` composes with resolved base system prompts.
- Define Go REPL interaction behavior for continuation steps.
- Define REPL subcall API modes used by harness recursion strategy.
- Define recursion depth and terminal-answer behavior at harness level.

## Non-Goals

- Defining provider transport behavior beyond `PRD-0008` gateway contracts.
- Defining sandbox hardening policy for Go REPL execution internals.
- Defining streaming inference behavior in v1.
- Defining cross-run prompt registries loaded from external files.

## Canonical Terminology

- `step`: one node-local decision cycle (`inference request -> structured
  model response -> decision handling`).
- `turn`: one transcript contribution. v1 turn roles are:
  - `user` (external request or harness-injected prompt/context feedback)
  - `model` (structured inference response)
- `action`: executable instruction emitted by a model `continue` decision.
- v1 continuation contract allows exactly one action per continue step.

## Harness Control Loop Contract

- Harness execution MUST begin with one root node at depth `0`.
- Each active node MUST run a control loop that requests structured inference
  with `schema_id=sigil.rlm.response.v1`.
- Node steps MUST interpret `validated_payload.decision` as follows:
  - `continue`: execute continuation behavior for that node.
  - `final`: complete that node with `validated_payload.final.answer`.
- Root-node `final` MUST terminate the run as successful completion.
- Harness MUST support both execution profiles selected by `rlm.enabled`:
  - `true`: recursive profile where `rlm_query` may create child nodes.
  - `false`: non-recursive profile where multi-step continuation is allowed but
    child node creation is disabled.

## System Prompt Resolution Contract

- Harness MUST resolve one base system prompt from a hard-coded provider map.
- v1 required hard-coded map entries:
  - `openai` => OpenAI system prompt.
  - `anthropic` => Anthropic system prompt.
- If provider has no dedicated prompt entry, harness MUST fallback to the
  OpenAI system prompt.
- Resolved prompt selection MUST be deterministic for equivalent provider input.

## System Prompt Append Contract

- `system_prompt_append` MUST default to empty string.
- If `system_prompt_append` is empty after trim, effective system prompt MUST be
  the resolved base system prompt.
- If `system_prompt_append` is non-empty after trim, effective system prompt
  MUST be:
  1. resolved base system prompt
  2. two newline characters
  3. raw configured append text
- Harness MUST apply the effective system prompt for root and recursive node
  inference steps.

## Go REPL Continuation Contract

- v1 harness continuation execution MUST use a Go REPL environment.
- Each node MUST have isolated REPL state.
- Continuation steps MUST consume structured response payload fields from
  `sigil.rlm.response.v1`.
- `decision=continue` MUST carry exactly one code action in
  `continuation.repl_code`.
- Harness MUST execute exactly one `continuation.repl_code` action per continue
  step in the same node REPL state.
- Enriched continuation branch fields (`continuation.intent`,
  `continuation.expected_observation`) are defined in `PRD-0014`.
- Harness MUST NOT rely on markdown code-fence parsing for continuation code.
- Full raw run context MUST remain REPL-local and MUST NOT be replayed in full
  to model-step inference requests.
- REPL execution outputs MUST be fed back into subsequent model steps as bounded
  feedback summaries, while full execution output remains persisted in action
  artifacts.
- If continuation payload is structurally invalid, harness step progression MUST
  fail with typed output-validation error propagation.
- Detailed REPL runtime contracts (engine/session boundary, fixed guardrails,
  import policy, artifact references, and typed REPL error taxonomy) are
  defined in `PRD-0010`.

## Recursive Subcall Contract

- Go REPL environment MUST expose node-local subcall functions:
  - `llm_query(prompt, context)` plain one-shot subcall
  - `rlm_query(prompt, context)` recursive subcall
  - `llm_query_batched(calls)` plain batched subcalls
  - `rlm_query_batched(calls)` recursive batched subcalls
- `llm_query` MUST execute as plain subcall inference and MUST NOT create child
  nodes.
- `rlm_query` MUST execute as child-node inference when recursion depth permits.
- Child node depth MUST be `parent_depth + 1`.
- Child node creation MUST be rejected when resulting depth would exceed
  `rlm.max_depth`.
- In recursive profile (`rlm.enabled=true`), max-depth subcalls MUST fallback to
  plain `llm_query`/`llm_query_batched` behavior rather than creating child
  nodes.
- Successful child-node `final` output MUST be returned to the caller context as
  `rlm_query` result.
- In non-recursive profile (`rlm.enabled=false`), `rlm_query` MUST remain bound
  and recursive subcall APIs (`rlm_query`, `rlm_query_batched`) MUST return
  typed depth-limit error behavior for all invocations and MUST NOT create child
  nodes.
- Subcall API execution modes, batched result typing, and detailed
  observability requirements are defined in `PRD-0013`.

## Completion and Failure Contract

- Run completion MUST require root-node `decision=final` with non-empty
  `final.answer`.
- Root finalization MUST validate resolvable `final.evidence[]` references
  before node/run completion as defined in `PRD-0014`.
- Unrecoverable inference, REPL, or harness orchestration errors MUST terminate
  run as `failed` with typed error metadata.
- Child-node failures MUST propagate deterministic failure information to caller
  node context unless policy explicitly escalates to run failure.

## Deferred Contracts

The following are explicitly deferred to future PRDs:

- Dynamic prompt registries sourced from files or remote stores.
- Provider-specific prompt versioning and migration workflows.
- Fine-grained REPL sandboxing and resource quotas.
- Multi-REPL language support beyond Go.
- Harness-level budget policies (for example max steps per node).

## Acceptance Scenarios

### Scenario SCN-0000: Resolves OpenAI base system prompt when llm.provider is openai

Given run configuration with `llm.provider=openai`  
When harness resolves base system prompt  
Then the OpenAI base system prompt is selected.

### Scenario SCN-0001: Resolves Anthropic base system prompt when llm.provider is anthropic

Given run configuration with `llm.provider=anthropic`  
When harness resolves base system prompt  
Then the Anthropic base system prompt is selected.

### Scenario SCN-0002: Falls back to OpenAI base system prompt when provider-specific prompt is not registered

Given run configuration with provider value lacking dedicated prompt mapping  
When harness resolves base system prompt  
Then the OpenAI base system prompt is selected as fallback.

### Scenario SCN-0003: Appends system_prompt_append to resolved base system prompt when append is non-empty

Given resolved base system prompt and non-empty `system_prompt_append`  
When effective system prompt is constructed  
Then effective prompt equals base prompt plus two newlines plus append text.

### Scenario SCN-0004: Uses resolved base system prompt unchanged when system_prompt_append is empty

Given resolved base system prompt and empty `system_prompt_append`  
When effective system prompt is constructed  
Then effective prompt equals resolved base prompt without appended suffix.

### Scenario SCN-0005: Starts harness execution with one root node at depth zero

Given a run enters `running` state with RLM enabled  
When harness execution starts  
Then exactly one root node is active at depth `0`.

### Scenario SCN-0006: Executes single continuation repl_code action from structured response in node-local REPL state

Given node continuation payload with non-empty `continuation.repl_code`  
When harness processes continuation step  
Then exactly one `continuation.repl_code` action executes in node-local REPL state.

### Scenario SCN-0007: Propagates typed output-validation error when continue payload omits continuation repl_code

Given node continuation payload with `decision=continue` and missing or empty `continuation.repl_code`  
When strict output validation runs  
Then continuation step fails with typed output-validation error.

### Scenario SCN-0008: Creates child node through rlm_query only when resulting depth does not exceed rlm.max_depth

Given active parent node and configured `rlm.max_depth`  
When `rlm_query` is invoked in node REPL context  
Then child node is created only if `parent_depth + 1 <= rlm.max_depth`.

### Scenario SCN-0009: Falls back to plain llm_query behavior when rlm_query call would exceed rlm.max_depth in recursive mode

Given active parent node at depth equal to `rlm.max_depth`  
When `rlm_query` is invoked in node REPL context  
Then no child node is created and plain llm_query fallback behavior is used.

### Scenario SCN-0010: Returns child final answer to caller REPL context on successful recursive subcall

Given child node inference reaches `decision=final`  
When child node completes  
Then child final answer is returned as `rlm_query` result to caller REPL context.

### Scenario SCN-0011: Completes run when root node emits decision final with non-empty final answer

Given root node inference output with `decision=final` and non-empty `final.answer`  
When harness evaluates root node step result  
Then run transitions to `completed` and terminal answer references root final output.

### Scenario SCN-0012: Defines step as one node-local decision cycle

Given an active node in harness execution  
When one inference request and its response handling complete  
Then exactly one node step is recorded for that decision cycle.

### Scenario SCN-0013: Records turns as user or model transcript contributions linked to a step

Given a node step in progress  
When transcript contributions are persisted for that step  
Then each contribution is recorded as a turn with role `user` or `model`.

### Scenario SCN-0014: Limits continue steps to exactly one executable action

Given a node step with `decision=continue`  
When continuation payload is validated  
Then exactly one executable action is accepted for that step.

### Scenario SCN-0015: Runs non-recursive multi-step profile when rlm.enabled is false and returns typed depth-limit feedback for recursive subcalls

Given harness execution with `rlm.enabled=false`  
When node steps continue and REPL code invokes `rlm_query` or `rlm_query_batched`  
Then harness executes multi-step continuation without creating child nodes  
And recursive subcalls return typed depth-limit feedback.

### Scenario SCN-0016: Allows multiple subcalls inside one continuation action while preserving one action per continue step

Given a node step with `decision=continue` and non-empty `continuation.repl_code`  
When repl_code execution issues multiple subcalls  
Then exactly one continuation action is recorded for that step and subcalls are observed inside that action.
