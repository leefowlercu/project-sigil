# WORKFLOW

## Delivery Flow

- Use Red, Green, Refactor for all behavioral changes.
- Red: add or update a failing test or acceptance scenario that captures intended behavior.
- Green: implement the smallest change needed to pass.
- Refactor: improve code and structure while keeping tests/scenarios green.

## Verification

- For superproject spec-only changes, verify internal consistency across ADR/PRD
  references and examples in each touched `docs/<subproject>/` tree.
- For `docs/sigil/` PRD structural changes, numbering changes, or traceability
  edits, run `./scripts/verify-specs --subproject sigil` and treat failures as
  blockers.
- For `docs/sigil-web/` PRD structural changes, numbering changes, traceability
  edits, or routed-state design-manifest edits, run
  `./scripts/verify-specs --subproject sigil-web` and treat failures as
  blockers.
- When a PRD refactor splits or replaces records, verify the corresponding PRD
  README index/migration map, `MATRIX.md`, ADR related-document links, and
  mapped acceptance titles in the same change.
- For submodule implementation verification, use the local workflow rules for
  the corresponding submodule (`sigil/` or `sigil-web/`) when present; otherwise
  follow that submodule's documented verification commands in `README.md`.
