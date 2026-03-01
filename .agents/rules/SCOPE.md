# SCOPE

## Repository Roles

- Treat this repository root as the source for behavioral specs and governance artifacts.
- Treat subproject specs as scoped under `docs/<subproject>/ADR` and `docs/<subproject>/PRD`.
- Treat `sigil/` and `sigil-web/` as implementation modules tracked as git submodules.

## Change Boundaries

- Spec-only tasks should primarily update the matching subproject docs folder
  under `docs/` plus root metadata files.
- Implementation changes belong in the corresponding submodule (`sigil/` or
  `sigil-web/`) and must follow that module's local agent rules when present.
