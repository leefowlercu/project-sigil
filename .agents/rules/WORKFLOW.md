# WORKFLOW

## Delivery Flow

- Use Red, Green, Refactor for all behavioral changes.
- Red: add or update a failing test or acceptance scenario that captures intended behavior.
- Green: implement the smallest change needed to pass.
- Refactor: improve code and structure while keeping tests/scenarios green.

## Verification

- For superproject spec-only changes, verify internal consistency across ADR/PRD
  references and examples in each touched `docs/<subproject>/` tree.
- For submodule implementation verification, use the local workflow rules for
  the corresponding submodule (`sigil/` or `sigil-web/`) when present; otherwise
  follow that submodule's documented verification commands in `README.md`.
