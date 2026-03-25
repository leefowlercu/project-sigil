# SPECS

## Source of Truth

- ADRs define architecture and tradeoffs.
- PRDs define product behavior; Gherkin acceptance criteria are authoritative
  for expected outcomes.
- Organize active specs under the owning subproject's `docs/<subproject>/ADR`
  and `docs/<subproject>/PRD` trees.

## Update Order

- When behavior changes, update or create PRD acceptance criteria first, then
  align implementation in the corresponding submodule.
- Keep ADRs current when architectural direction changes.

## Consistency

- Keep terminology consistent across `README.md`, ADRs, PRDs, and submodule
  docs.
- Prefer concrete, testable acceptance language over implementation detail.

## File Naming

- PRD files **MUST** use the naming pattern
  `PRD-<4-digit>-<slug>-specification.md`.

## Acceptance Traceability

- Every PRD acceptance scenario **MUST** map to at least one acceptance
  scenario in the corresponding submodule.
- Every PRD acceptance scenario **MUST** include a scenario ID in the form
  `SCN-xxxx`.
- Scenario IDs **MUST** use zero-padded 4-digit integers and be unique within a
  PRD.
- Scenario IDs **MUST** increment from `SCN-0000` within each PRD.
- PRD scenario titles **SHOULD** match the mapped acceptance scenario title
  exactly to keep traceability mechanical and reviewable.
- PRD scenario titles **SHOULD** be globally unique within a subproject so
  title-based traceability remains unambiguous.
- When multiple PRDs touch the same behavior, exactly one PRD **SHOULD** own
  the acceptance scenario and the others **SHOULD** reference that owner
  instead of duplicating the scenario.
- Subproject `MATRIX.md` files **MUST** include a dedicated scenario ID column.
- `MATRIX.md` scenario IDs **MUST** match PRD scenario IDs exactly for mapped
  scenarios.
- `sigil` traceability map: `docs/sigil/PRD/MATRIX.md` ->
  `sigil/acceptance/features/harness.feature`.
- When behavior changes, update PRD acceptance criteria and the matching
  subproject traceability map before or alongside implementation updates.
- Existing scenario IDs **SHOULD** remain stable over time; append new IDs
  instead of renumbering unless a deliberate migration is documented.
