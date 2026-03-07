# AGENTS.md

This superproject is the specification and governance layer for the Siĝil harness ecosystem. Specifications are organized per subproject under `docs/<subproject>/ADR` and `docs/<subproject>/PRD`. Production implementations live in git submodules and must stay aligned with these specs.

## Table of Contents

- [Repository Model](#repository-model)
- [Workflow Rules](#workflow-rules)
- [Specification Rules](#specification-rules)
- [Submodule Rules](#submodule-rules)
- [Change Control Rules](#change-control-rules)
- [Reference Documents](#reference-documents)

## Repository Model

- Follow repository boundaries and ownership rules in [`SCOPE`](.agents/rules/SCOPE.md).

## Workflow Rules

- Use the delivery flow in [`WORKFLOW`](.agents/rules/WORKFLOW.md) for superproject governance/spec work.
- Before broad PRD refactors or renumbering work, verify the current subproject docs state first instead of refactoring blind.
- For work inside `sigil/`, follow submodule-local instructions in [`sigil/AGENTS.md`](sigil/AGENTS.md) when present; otherwise use `sigil/README.md` plus any local `.agents/rules/` files.
- For work inside `sigil-web/`, follow submodule-local instructions in [`sigil-web/AGENTS.md`](sigil-web/AGENTS.md) when present; otherwise use `sigil-web/README.md` plus any local `.agents/rules/` files.

## Specification Rules

- For ADR/PRD updates and acceptance-source behavior, use [`SPECS`](.agents/rules/SPECS.md).
- Use subproject-specific `MATRIX` files as acceptance traceability contracts.
- PRD files MUST use the naming pattern `PRD-<4-digit>-<slug>-specification.md`.
- Organize PRDs by durable subsystem, feature-family, or mechanism ownership rather than delivery chronology.
- Each behavior SHOULD have one normative owning PRD; when scope overlaps, reference the owner instead of duplicating acceptance criteria across PRDs.
- `sigil`: [`docs/sigil/PRD/MATRIX.md`](docs/sigil/PRD/MATRIX.md) maps PRD scenarios to `sigil/acceptance/features/harness.feature`.
- `sigil-web`: [`docs/sigil-web/PRD/MATRIX.md`](docs/sigil-web/PRD/MATRIX.md) maps PRD scenarios to `sigil-web/acceptance/features/web_ui.feature`.
- PRD acceptance scenarios MUST include scenario IDs in the form `SCN-xxxx` (zero-padded, incrementing from `SCN-0000` within each PRD).
- Matching `MATRIX` rows MUST include the same scenario IDs and keep a dedicated scenario ID column.
- When adding or changing PRD acceptance criteria, update PRD scenarios, the matching subproject `MATRIX`, and the corresponding submodule acceptance scenario in the same change.
- Keep PRD scenario IDs and titles mechanically aligned with mapped acceptance scenarios unless there is a documented exception.
- When splitting, replacing, or renumbering PRDs, update the subproject PRD README index/migration map, mapped acceptance titles, and ADR backlinks in the same change.
- For `sigil` PRD work, follow the taxonomy and lifecycle rules in [`docs/sigil/PRD/README.md`](docs/sigil/PRD/README.md) and run `./scripts/verify-specs --subproject sigil` before concluding.

## Submodule Rules

- For work that touches `sigil` or `sigil-web`, follow [`SUBMODULE`](.agents/rules/SUBMODULE.md).

## Change Control Rules

- For commit hygiene and submodule pointer updates, use [`GIT`](.agents/rules/GIT.md).
- Commits and pushes require explicit user authorization in the current conversation; do not assume publish intent.

## Reference Documents

- Superproject overview: [`README.md`](README.md)
- Documentation layout: [`docs/README.md`](docs/README.md)
- Sigil architecture decisions: [`docs/sigil/ADR`](docs/sigil/ADR)
- Sigil product requirements: [`docs/sigil/PRD`](docs/sigil/PRD)
- Sigil PRD taxonomy and lifecycle rules: [`docs/sigil/PRD/README.md`](docs/sigil/PRD/README.md)
- Sigil acceptance traceability contract: [`docs/sigil/PRD/MATRIX.md`](docs/sigil/PRD/MATRIX.md)
- Sigil PRD verifier: [`scripts/verify-specs`](scripts/verify-specs)
- Sigil-web architecture decisions: [`docs/sigil-web/ADR`](docs/sigil-web/ADR)
- Sigil-web product requirements: [`docs/sigil-web/PRD`](docs/sigil-web/PRD)
- Sigil-web acceptance traceability contract: [`docs/sigil-web/PRD/MATRIX.md`](docs/sigil-web/PRD/MATRIX.md)
- Sigil submodule overview: [`sigil/README.md`](sigil/README.md)
- Sigil submodule agent instructions (when present): [`sigil/AGENTS.md`](sigil/AGENTS.md)
- Sigil-web submodule overview: [`sigil-web/README.md`](sigil-web/README.md)
- Sigil-web submodule agent instructions (when present): [`sigil-web/AGENTS.md`](sigil-web/AGENTS.md)
