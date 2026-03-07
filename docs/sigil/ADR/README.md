# ADR

This directory stores architecture decision records for the `sigil` subproject.

- Use ADRs to capture long-lived technical decisions and tradeoffs.
- Cross-reference related PRDs and submodule changes when behavior is affected.

## Status

- [`ADR-0001-cobra-cli-framework.md`](ADR-0001-cobra-cli-framework.md): Use
  Cobra for Sigil CLI
- [`ADR-0002-viper-typed-configuration.md`](ADR-0002-viper-typed-configuration.md):
  Use typed Viper configuration
- [`ADR-0003-godog-bdd-acceptance-testing.md`](ADR-0003-godog-bdd-acceptance-testing.md):
  Use Godog for BDD acceptance testing
- [`ADR-0004-slog-structured-json-file-logging.md`](ADR-0004-slog-structured-json-file-logging.md):
  Use slog for structured JSON file logging
- [`ADR-0005-sigil-event-sourced-run-architecture.md`](ADR-0005-sigil-event-sourced-run-architecture.md):
  Use event-sourced architecture for sigil run runtime state
- [`ADR-0006-sigil-run-node-identity-model.md`](ADR-0006-sigil-run-node-identity-model.md):
  Define sigil run/node identity model
- [`ADR-0007-sigil-llm-gateway-abstraction.md`](ADR-0007-sigil-llm-gateway-abstraction.md):
  Define gateway abstraction pattern for sigil inference
- [`ADR-0008-sigil-openrouter-responses-gateway.md`](ADR-0008-sigil-openrouter-responses-gateway.md):
  Define OpenRouter Responses API as sigil v1 inference gateway
- [`ADR-0009-sigil-go-repl-engine-and-runtime-boundary.md`](ADR-0009-sigil-go-repl-engine-and-runtime-boundary.md):
  Define Go REPL engine architecture and runtime boundary for sigil v1
- [`ADR-0010-sigil-run-start-blocking-harness-entrypoint.md`](ADR-0010-sigil-run-start-blocking-harness-entrypoint.md):
  Define sigil run start as synchronous harness execution entrypoint
- [`ADR-0011-sigil-harness-model-input-boundary-and-message-contract.md`](ADR-0011-sigil-harness-model-input-boundary-and-message-contract.md):
  Define harness model-input boundary and message contract for bounded step input
- [`ADR-0012-sigil-repl-subcall-modes-batching-and-observability.md`](ADR-0012-sigil-repl-subcall-modes-batching-and-observability.md):
  Define REPL subcall API modes, batching behavior, and subcall observability contracts
- [`ADR-0013-sigil-structured-output-verifiability-and-prompt-schema-sync.md`](ADR-0013-sigil-structured-output-verifiability-and-prompt-schema-sync.md):
  Define enriched structured-output verifiability and runtime prompt-schema synchronization
- [`ADR-0014-sigil-non-timeout-stability-and-failure-observability-hardening.md`](ADR-0014-sigil-non-timeout-stability-and-failure-observability-hardening.md):
  Define node-failure observability and non-timeout inference/compile hardening
- [`ADR-0015-sigil-deterministic-runtime-governance-guardrails.md`](ADR-0015-sigil-deterministic-runtime-governance-guardrails.md):
  Define deterministic runtime governance guardrails for harness execution
- [`ADR-0016-sigil-accounting-ledger-and-provenance-contract.md`](ADR-0016-sigil-accounting-ledger-and-provenance-contract.md):
  Define accounting ledger, provenance, and rollup contracts for sigil runs
- [`ADR-0017-sigil-local-run-stop-pid-signal-control.md`](ADR-0017-sigil-local-run-stop-pid-signal-control.md):
  Define local PID-and-signal graceful-stop control for sigil CLI runs

## Next ADR

Create the next record as:

- `docs/sigil/ADR/ADR-0018-<slug>.md`
