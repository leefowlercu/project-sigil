# PRD

This directory stores product requirement documents for the `sigil` subproject.

- PRDs define expected external behavior and acceptance criteria.
- Organize PRDs by stable subsystem, feature, or mechanism ownership, not by delivery chronology.
- Prefer one normative owner for a behavior; other PRDs should reference that owner instead of restating the contract.
- Update PRD acceptance scenarios before or alongside submodule implementation changes.
- Use scenario IDs in the form `SCN-xxxx` for PRD acceptance scenarios.
- Use PRD filenames in the form `PRD-<4-digit>-<slug>-specification.md`.
- Keep PRD scenario IDs and titles aligned with [`docs/sigil/PRD/MATRIX.md`](MATRIX.md).

## Management Model

- Keep PRDs behavior-centric and acceptance-backed.
- Keep each PRD focused on one durable concern boundary:
  - subsystem
  - feature family
  - mechanism
- Prefer adding a new PRD over growing an existing PRD across multiple ownership boundaries.
- When a PRD is replaced or split, update this README and the traceability matrix in the same change.
- Reuse scenario titles when behavior is unchanged so acceptance traceability stays mechanical.

## Lifecycle and Verification

- Keep PRD acceptance scenario titles globally unique within `sigil`.
- If multiple PRDs need to mention the same behavior, keep the acceptance scenario in one owning PRD and reference that owner from the others.
- When a PRD is split, replaced, or retired, update the legacy migration map and affected ADR backlinks in the same change.
- Run `./scripts/verify-specs --subproject sigil` after structural PRD, matrix, or acceptance-title edits.

## Numbering Blocks

- `PRD-0100` to `PRD-0199`: configuration and CLI surface
- `PRD-0200` to `PRD-0299`: runtime lifecycle and eventing
- `PRD-0300` to `PRD-0399`: inference, schemas, and prompt composition
- `PRD-0400` to `PRD-0499`: harness and REPL execution
- `PRD-0500` to `PRD-0599`: operational policy and accounting

## Current PRDs

### Configuration and CLI

- [`PRD-0100-sigil-application-config-specification.md`](PRD-0100-sigil-application-config-specification.md):
  Sigil application configuration contract
- [`PRD-0110-sigil-cli-surface-specification.md`](PRD-0110-sigil-cli-surface-specification.md):
  Sigil CLI surface contract
- [`PRD-0120-sigil-run-config-core-specification.md`](PRD-0120-sigil-run-config-core-specification.md):
  Sigil run-configuration core contract
- [`PRD-0130-sigil-run-start-command-inputs-specification.md`](PRD-0130-sigil-run-start-command-inputs-specification.md):
  Sigil run-start command input contract
- [`PRD-0140-sigil-application-logging-specification.md`](PRD-0140-sigil-application-logging-specification.md):
  Sigil application logging contract

### Runtime Core

- [`PRD-0200-sigil-run-lifecycle-state-machine-specification.md`](PRD-0200-sigil-run-lifecycle-state-machine-specification.md):
  Sigil run lifecycle state machine contract
- [`PRD-0210-sigil-run-event-contract-specification.md`](PRD-0210-sigil-run-event-contract-specification.md):
  Sigil run event envelope, catalog, ordering, and durable storage contract

### Inference and Schemas

- [`PRD-0300-sigil-inference-gateway-specification.md`](PRD-0300-sigil-inference-gateway-specification.md):
  Sigil inference gateway and response-normalization contract
- [`PRD-0310-sigil-structured-response-schema-and-evidence-specification.md`](PRD-0310-sigil-structured-response-schema-and-evidence-specification.md):
  Sigil structured response schema and evidence contract
- [`PRD-0320-sigil-prompt-composition-and-schema-synchronization-specification.md`](PRD-0320-sigil-prompt-composition-and-schema-synchronization-specification.md):
  Sigil provider prompt composition and runtime schema-synchronization contract

### Harness and REPL Execution

- [`PRD-0400-sigil-harness-control-loop-specification.md`](PRD-0400-sigil-harness-control-loop-specification.md):
  Sigil harness control-loop contract
- [`PRD-0410-sigil-run-start-command-execution-specification.md`](PRD-0410-sigil-run-start-command-execution-specification.md):
  Sigil run-start command execution contract
- [`PRD-0420-sigil-model-input-boundary-and-step-feedback-specification.md`](PRD-0420-sigil-model-input-boundary-and-step-feedback-specification.md):
  Sigil model-input boundary and step-feedback contract
- [`PRD-0430-sigil-go-repl-runtime-specification.md`](PRD-0430-sigil-go-repl-runtime-specification.md):
  Sigil Go REPL runtime contract
- [`PRD-0440-sigil-repl-subcall-apis-and-batching-specification.md`](PRD-0440-sigil-repl-subcall-apis-and-batching-specification.md):
  Sigil REPL subcall API and batching contract

### Operational Policy and Accounting

- [`PRD-0500-sigil-deterministic-runtime-guardrails-specification.md`](PRD-0500-sigil-deterministic-runtime-guardrails-specification.md):
  Sigil deterministic runtime guardrails contract
- [`PRD-0510-sigil-accounting-ledger-specification.md`](PRD-0510-sigil-accounting-ledger-specification.md):
  Sigil accounting ledger contract

## Legacy Migration Map

- `PRD-0001` -> `PRD-0100`
- `PRD-0002` -> `PRD-0110`
- `PRD-0003` -> `PRD-0120`, `PRD-0300`, `PRD-0500`, `PRD-0510`
- `PRD-0004` -> `PRD-0130`
- `PRD-0005` -> `PRD-0140`
- `PRD-0006` -> `PRD-0200`
- `PRD-0007` -> `PRD-0210`, `PRD-0500`, `PRD-0510`
- `PRD-0008` -> `PRD-0300`, `PRD-0310`
- `PRD-0009` -> `PRD-0320`, `PRD-0400`, `PRD-0440`
- `PRD-0010` -> `PRD-0420`, `PRD-0430`, `PRD-0440`
- `PRD-0011` -> `PRD-0410`
- `PRD-0012` -> `PRD-0420`
- `PRD-0013` -> `PRD-0440`, `PRD-0510`
- `PRD-0014` -> `PRD-0310`, `PRD-0320`
- `PRD-0015` -> distributed into `PRD-0210`, `PRD-0300`, `PRD-0420`, and `PRD-0430`
- `PRD-0016` -> `PRD-0500`
- `PRD-0017` -> `PRD-0510`

## Next PRD

Create the next record within the correct subsystem block using the first open
number in that block.

Examples:

- `docs/sigil/PRD/PRD-0150-<slug>-specification.md` for a new CLI/config contract
- `docs/sigil/PRD/PRD-0330-<slug>-specification.md` for a new inference contract
- `docs/sigil/PRD/PRD-0450-<slug>-specification.md` for a new harness/REPL contract
