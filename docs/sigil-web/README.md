# Sigil-Web Docs

This directory stores the active superproject ADR, PRD, and acceptance
traceability records for the `sigil-web` subproject.

## Structure

- `ADR/`: architectural decisions and long-lived technical tradeoffs
- `PRD/`: behavior specifications and acceptance traceability

`sigil-web` follows the same spec-driven workflow as `sigil`:

1. update or add the owning PRD acceptance scenarios
2. keep `docs/sigil-web/PRD/MATRIX.md` aligned with those scenarios
3. update the mapped acceptance feature titles in `sigil-web/acceptance/`
4. implement the smallest change needed in the `sigil-web/` submodule
5. run `./scripts/verify-specs --subproject sigil-web`
