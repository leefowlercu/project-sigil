# PRD

This directory stores product requirement documents for the `sigil` subproject.

- PRDs define expected external behavior and acceptance criteria.
- Update PRD acceptance scenarios before or alongside submodule implementation changes.
- Use scenario IDs in the form `SCN-xxxx` for PRD acceptance scenarios.
- Use PRD filenames in the form `PRD-<4-digit>-<slug>-specification.md`.
- Keep PRD scenario IDs and titles aligned with `docs/sigil/PRD/MATRIX.md` mappings.

## Status

- [`PRD-0001-sigil-config-specification.md`](PRD-0001-sigil-config-specification.md):
  Sigil application configuration contract
- [`PRD-0002-sigil-cli-surface-specification.md`](PRD-0002-sigil-cli-surface-specification.md):
  Sigil CLI surface contract
- [`PRD-0003-sigil-run-config-specification.md`](PRD-0003-sigil-run-config-specification.md):
  Sigil run configuration contract
- [`PRD-0004-sigil-run-start-cli-subcommand-specification.md`](PRD-0004-sigil-run-start-cli-subcommand-specification.md):
  Sigil run start CLI subcommand contract
- [`PRD-0005-sigil-application-logging-specification.md`](PRD-0005-sigil-application-logging-specification.md):
  Sigil application logging contract
- [`PRD-0006-sigil-run-lifecycle-state-machine-specification.md`](PRD-0006-sigil-run-lifecycle-state-machine-specification.md):
  Sigil run lifecycle state machine contract
- [`PRD-0007-sigil-run-event-contract-specification.md`](PRD-0007-sigil-run-event-contract-specification.md):
  Sigil run event envelope and durable storage contract
- [`PRD-0008-sigil-llm-inference-via-gateway-specification.md`](PRD-0008-sigil-llm-inference-via-gateway-specification.md):
  Sigil LLM inference via gateway contract

## Next PRD

Create the next record as:

- `docs/sigil/PRD/PRD-0009-<slug>-specification.md`
